Como diz o Uncle Bob em [[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)]:

> *"[...] os sistemas mais flexíveis são aqueles em que as dependências de código-fonte se referem apenas a abstrações e não a itens concretos."*

Isto quer dizer basicamente que devemos depender de abstrações e não implementações concretas.

**Explicação:**
- Módulos de alto nível não devem depender de módulos de baixo nível. ***Ambos devem depender de abstrações.***
- Abstrações não devem depender de detalhes. ***Detalhes devem depender de abstrações***

**Use case - Sistema de notificação**
```dart title:notification_controller.dart
// VIOLACAO DIP - dependencia direta de implementacao concreta
class NotificationController {
	// dependencia concreta
	final EmailService emailService = EmailService ();
	
	Future<void> notifyUser(String email, String message) async {
		// Acoplado diretamente a EmailService
		await emailService.send(email, message);
	}
}

/*
	Problemas:
	1. Impossivel testar sem enviar emails reais
	2. Impossivel trocar para SMS sem modificar NotificationCoontroller
	3. NotificationController conhece detalhes de implementacao
*/
```

**SOLUÇÃO:**
```dart title:notification_service.dart
abstract class NotificationService {
	Future<void> send(String recipient, String message);
}
```

```dart title:email_service_interface.dart
class EmailService implements NotificationService {
	@override 
	Future<void> send(String recipient, String message) async {
		// Implementacao
	}
}
```

```dart title:sms_service_interface.dart
class SMSService implements NotificationService {
	@override 
	Future<void> send(String recipient, String message) async {
		// Implementacao
	}
}
```

```dart title:fake_notification_service.dart
class FakeNotificationService implements NotificationService {
	List<String> sentMessages = [];
	@override 
	Future<void> send(String recipient, String message) async {
		sentMessages.add('$recipient: $message');
	}
}
```

> Agora a classe `NotificationController` depende de ABSTRAÇÃO:

```dart title:notification_controller.dart
class NotificationController {
	final NotificationService service; // <- Abstracao, nao implementacao
	
	NotificationController(this.service) // <- Injecao de dependencia (geralmente feita via constructor)
	
	Future<void> notifyUser(String recipient, String message) async {
		await service.send(recipient, message);
	}
	
}
```

> Utilização em produção:
> 
```dart title:main.dart
final controller = NotificationController(EmailService());

// Uso em testes
final testController = NotificationController(FakeNotificationService());

// facil trocar de implementacao
final smsController = NotificationController(SMSService());
```

**Diagrama de dependências:**

- Antes (VIOLA DIP):

```markdown title:dependencia_de_servico_concreto.md
[NotificationController] --depende--> [Email Service (concreto)]
		↑                                      ↑
    Alto nível                            Baixo nível
```

- Depois (RESPEITA DIP):

```markdown title:depende_de_abstracao.md
[NotificationController] --depende--> [NotificationService (abstração)]
                                              ↑ implementa
                                       [EmailService (concreto)]
        ↑                                      ↑
    Alto nível                            Baixo nível
    
    Ambos dependem da abstração!
```

**Benefícios:**
- Fácil substituir implementações
- Testável (usa mocks/fakes)
- Módulos desacoplados
- Flexível para mudanças