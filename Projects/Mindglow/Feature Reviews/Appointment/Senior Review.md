# Senior Code Review — ZEGOCLOUD Appointment Feature

> **Scope:** Revisão completa das Fases 1–4, com foco no erro crítico da Fase 3 e na estratégia de correção correta.

---

## Veredicto por Fase

| Fase | Estado | Avaliação |
|---|---|---|
| Fase 1 — Infraestrutura ZEGO | ✅ Implementada | Sólida, com uma nota menor |
| Fase 2 — Domain + Data Layer | ✅ Implementada | Muito boa — padrão correto |
| Fase 3 — Integração UI | ❌ **QUEBRADA** | Erro crítico de API incorreta |
| Fase 4 — Pós-Sessão + Guia | ⚠️ Parcial | Existe, mas tem falha de segurança |

---

## 🔴 FASE 3 — Problema Crítico: API Inexistente

### O que está errado em `call_session_wrapper.dart`

O ficheiro usa **duas APIs que não existem** no pacote `zego_uikit`:

| Linha | Código Escrito | Problema |
|---|---|---|
| 20 | `StreamSubscription<ZegoCallState>` | `ZegoCallState` **não existe** no SDK |
| 28 | `ZegoUIKit().getCallStateStream()` | Método **não existe** em `ZegoUIKit` |
| 29, 32 | `ZegoCallState.connected` / `.ended` | Enums **não existem** |

### Raíz do Problema — Diagnóstico de Arquitetura

O `CallSessionWrapper` foi desenhado com base em lógica imaginada — uma API de stream para estado de chamada que **nunca existiu** no UIKit. A forma oficial do ZEGOCLOUD para tracking de chamada é completamente diferente:

```
❌ Abordagem errada (inexistente):
   ZegoUIKit().getCallStateStream() → stream de estados

✅ Abordagem correta (documentada):
   config.duration.onDurationUpdate → callback por segundo
   config.events.onCallEnd          → callback quando a chamada termina
```

Ambos estão no `requireConfig` callback dentro de `ZegoUIKitPrebuiltCallInvitationService().init()`, já existente no `ZegoService`.

### Problema Secundário — Mapeamento callID → appointmentId

O código tem este comentário revelador:
```dart
// Nota: Numa integração real, teríamos de associar o roomId ao Appointment.id
```

Este não é um detalhe — **é um bloqueio funcional**. Quando a chamada termina, o ZEGO fornece o `callID` (definido pelo especialista), mas o BLoC precisa do `appointment.id` (UUID do Supabase). Sem este mapeamento, `AppointmentCallEnded` nunca conseguirá actualizar o appointment correto.

**Solução:** O especialista deve incluir o `appointment.id` no campo `customData` quando envia o `ZegoSendCallInvitationButton`. O paciente recebe-o em `onIncomingCallReceived` e deve guardá-lo.

---

## Estratégia de Correção Correta

### Princípio: Mover o tracking para dentro do `ZegoService`

O `CallSessionWrapper` tentou ser um observador externo — mas o ZEGO não expõe streams públicos. A solução correcta é:

1. **`ZegoService`** expõe um `Stream` próprio de fim de chamada
2. O tracking interno usa `config.duration.onDurationUpdate` + `config.events.onCallEnd`
3. O `CallSessionWrapper` subscribe ao stream do `ZegoService` (sem depender de APIs internas do ZEGO)

### Diagrama do Fluxo Correto

```
Especialista envia invitation (customData = appointmentId)
       ↓
ZegoService.invitationEvents.onIncomingCallReceived
  → guarda appointmentId do customData
       ↓
Chamada aceite e conectada
  → config.duration.onDurationUpdate guarda duração em andamento
       ↓
Utilizador pressiona Hang Up
  → config.events.onCallEnd dispara
  → ZegoService._callEndedController.add(...)
       ↓
CallSessionWrapper ouve ZegoService.onCallEnded
  → context.read<AppointmentsBloc>().add(AppointmentCallEnded(...))
```

---

## Análise das Outras Fases

### ✅ Fase 1 — Infraestrutura ZEGO

**Pontos fortes:**
- `ZegoService.bootstrap()` chamado **antes** de `runApp()` — correto
- `navigatorKey` partilhado corretamente entre `ZegoService` e `GoRouter`
- `requireConfig` com lógica 1-on-1 vs group correcta
- Preparação para token auth está bem estruturada (placeholder comentado)
- `_isInitialized` guard evita double-init

**Nota menor:**
O callback `onIncomingCallReceived` tem `ZegoCallInvitationType callType` como parâmetro. A documentação oficial atual usa `ZegoCallType callType`. Dependendo da versão do pacote, isto pode gerar um warning. Verificar com `flutter analyze`.

---

### ✅ Fase 2 — Domain + Data Layer

**Pontos fortes:**
- `AppointmentEntity` correto: campos ZEGO (`roomId`, `actualStartTime`, etc.) bem separados com comentário
- `AppointmentStatus.fromString()` com fallback `unknown` — pattern robusto
- `appointments_remote_datasource.dart`: query com join `*, specialists(*)` é eficiente
- `appointments_repository_impl.dart`: error handling correto com `Either<Failure, T>`
- BLoC com **optimistic update** em `_onAppointmentCallEnded` — excelente padrão UX

**Problema menor — falha silenciosa:**
```dart
// Em appointments_bloc.dart, linha 94-102:
await _updateCallMetrics(...);
// Resultado nunca é verificado — falha de rede ignorada silenciosamente
```
Se o `updateCallMetrics` falhar (rede cortada logo após a chamada), o paciente fica com UI actualizada mas Supabase desactualizado. Deveria emitir pelo menos um log ou estado de warning.

---

### ⚠️ Fase 4 — Pós-Sessão + Guia

**`zego_integration_guide.md` — Falha de Segurança:**

```markdown
# Linha 8 do guia:
- **AppSign**: `5a931f7ca84e9035e7626e4e2ff42e9737c3682634826798837a110fb3eaae0f`
```

> [!CAUTION]
> **AppSign exposto em texto claro.** Se este repositório for público ou partilhado, as credenciais ZEGO estão comprometidas. O AppSign com o AppID permite criar chamadas em nome do projeto, gerando custos. **Remover imediatamente** e substituir por `<YOUR_APP_SIGN>` ou referenciar o `.env`.

---

## Plano de Correção Detalhado

### Passo 1 — Adicionar stream de fim de chamada ao `ZegoService`

```dart
// Em zego_service.dart — adicionar:

import 'dart:async';

/// Dados emitidos quando uma chamada termina
class CallEndedData {
  final String appointmentId; // vem do customData do especialista
  final DateTime startTime;
  final DateTime endTime;
  final int durationMinutes;

  CallEndedData({
    required this.appointmentId,
    required this.startTime,
    required this.endTime,
    required this.durationMinutes,
  });
}

// Dentro da classe ZegoService:
final _callEndedController = StreamController<CallEndedData>.broadcast();
Stream<CallEndedData> get onCallEnded => _callEndedController.stream;

// Variáveis internas de tracking:
String? _pendingAppointmentId;
DateTime? _callStartTime;
Duration _lastDuration = Duration.zero;
```

### Passo 2 — Actualizar `requireConfig` e `invitationEvents` no `ZegoService`

```dart
// Em invitationEvents, capturar o appointmentId do customData:
onIncomingCallReceived: (
  String callID,
  ZegoCallUser caller,
  ZegoCallInvitationType callType,
  List<ZegoCallUser> callees,
  String customData,        // ← O especialista coloca o appointmentId aqui
) {
  _pendingAppointmentId = customData; // guardar para quando a chamada terminar
  debugPrint('[ZegoService] 📞 Chamada recebida | appointmentId: $customData');
},

// Em requireConfig, substituir a config.duration e adicionar events:
config.duration.isVisible = true;
config.duration.onDurationUpdate = (Duration duration) {
  _lastDuration = duration;
  if (_callStartTime == null && duration.inSeconds >= 1) {
    _callStartTime = DateTime.now().subtract(duration);
  }
};

config.events.onCallEnd = (ZegoCallEndEvent event, VoidCallback defaultAction) {
  final endTime = DateTime.now();
  final start = _callStartTime ?? endTime;
  
  if (_pendingAppointmentId != null) {
    _callEndedController.add(CallEndedData(
      appointmentId: _pendingAppointmentId!,
      startTime: start,
      endTime: endTime,
      durationMinutes: _lastDuration.inMinutes,
    ));
  }
  
  // Reset state
  _callStartTime = null;
  _pendingAppointmentId = null;
  _lastDuration = Duration.zero;
  
  defaultAction(); // Deixar o ZEGO fechar a UI
};
```

### Passo 3 — Reescrever `CallSessionWrapper` para usar o stream correto

```dart
// call_session_wrapper.dart — versão corrigida:
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:mindglow/core/services/zego_service.dart';
import 'package:mindglow/features/appointments/presentation/bloc/appointments_bloc.dart';

class CallSessionWrapper extends StatefulWidget {
  final Widget child;
  const CallSessionWrapper({super.key, required this.child});

  @override
  State<CallSessionWrapper> createState() => _CallSessionWrapperState();
}

class _CallSessionWrapperState extends State<CallSessionWrapper> {
  StreamSubscription<CallEndedData>? _sub;

  @override
  void initState() {
    super.initState();
    _sub = ZegoService().onCallEnded.listen((data) {
      if (context.mounted) {
        context.read<AppointmentsBloc>().add(
          AppointmentCallEnded(
            id: data.appointmentId,
            startTime: data.startTime,
            endTime: data.endTime,
            durationMinutes: data.durationMinutes,
          ),
        );
        // Navegar para pós-sessão (Fase 4)
        // context.push('/consultations/post-session/${data.appointmentId}');
      }
    });
  }

  @override
  void dispose() {
    _sub?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => widget.child;
}
```

### Passo 4 — Guia do especialista

```dart
// Na app do especialista, ao enviar a chamada:
ZegoSendCallInvitationButton(
  isVideoCall: true,
  customData: appointment.id, // ← CRÍTICO: o paciente usará isto para associar ao appointment
  invitees: [
    ZegoUIKitUser(id: patient.supabaseId, name: patient.name),
  ],
  resourceID: 'mindglow_call',
)
```

---

## Checklist de Acções Imediatas

- [x] **[CRÍTICO]** Reescrever `call_session_wrapper.dart` com a API correta
- [x] **[CRÍTICO]** Adicionar `StreamController<CallEndedData>` ao `ZegoService`
- [x] **[CRÍTICO]** Adicionar `config.events.onCallEnd` em `requireConfig`
- [x] **[CRÍTICO]** Capturar `customData` em `onIncomingCallReceived`
- [ ] **[SEGURANÇA]** Remover AppSign exposto do `zego_integration_guide.md`
- [ ] **[MINOR]** Verificar callback type: `ZegoCallInvitationType` vs `ZegoCallType`
- [ ] **[MINOR]** Tratar falha de `updateCallMetrics` no BLoC
- [ ] **[GUIA]** Documentar `customData: appointment.id` para app do especialista
- [ ] Correr `flutter analyze` após as correções

---

## Avaliação Global

O plano de implementação original **está conceitualmente correcto** — as fases fazem sentido, a clean architecture foi respeitada, o BLoC está bem estruturado, e a estratégia de "apenas receber" na app do paciente é a decisão certa.

O problema foi **exclusivamente na execução da Fase 3**: a API do ZEGOCLOUD UIKit não expõe streams públicos de estado de chamada. A abordagem correcta (e documentada) é usar os callbacks do `requireConfig` — que já existem no `ZegoService` — em vez de tentar observar o estado externamente.

A boa notícia: a correcção é **cirúrgica**. Não requer mudanças de arquitectura, apenas mover a lógica de tracking para o sítio correcto e reescrever o wrapper.