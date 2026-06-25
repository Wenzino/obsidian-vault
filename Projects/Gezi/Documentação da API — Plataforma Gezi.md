### Backend FastAPI · Arquitectura Event-Driven · v1.0

**Projecto:** Gezi — Modernização do Sistema CREDELEC via IoT **Responsável:** Dai Wen Xuan · Universidade Zambeze · 2026

---

## 1. Visão Geral

A API segue **Clean Architecture** (RNF05) com separação por módulos de domínio:

```
app/
├── modules/
│   ├── auth/         # RF02, RN04
│   ├── meter/        # RF01
│   ├── recharge/      # RF03, RF16, RN01, RN10
│   ├── payment/       # RF04, RNF06
│   └── iot/           # RF05, RF11, RN05, RN06, RN07
└── core/
    ├── security/      # JWT, HMAC, TLS
    ├── mqtt/          # cliente MQTT partilhado
    └── supabase/      # cliente BD partilhado
```

**Base URL:** `https://api.gezi.mz/v1` _(Railway — produção sandbox)_

**Autenticação:** Bearer Token (JWT) em todos os endpoints excepto `/auth/*` e callbacks de pagamento.

```http
Authorization: Bearer <jwt_token>
```

**Formato de resposta padrão:**

```json
{
  "success": true,
  "data": { },
  "error": null,
  "timestamp": "2026-06-25T14:30:00Z"
}
```

---

## 2. Módulo `auth` — Autenticação

_Cobre: RF02, RN04_

### `POST /auth/otp/request`

Solicita o envio de um código OTP por SMS para o número de telemóvel indicado.

**Request Body**

```json
{
  "phone_number": "+258841234567"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "otp_sent": true,
    "expires_in_seconds": 300
  }
}
```

**Erros**

|Código|Descrição|
|---|---|
|`400`|Número de telemóvel inválido|
|`429`|Demasiados pedidos OTP — _rate limit_|

---

### `POST /auth/otp/verify`

Valida o código OTP recebido e devolve o JWT de sessão.

**Request Body**

```json
{
  "phone_number": "+258841234567",
  "otp_code": "482917"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJI...",
    "token_type": "bearer",
    "expires_in": 3600,
    "user_id": "usr_8f3a2b",
    "is_new_user": false
  }
}
```

**Erros**

|Código|Descrição|
|---|---|
|`401`|Código OTP inválido ou expirado|
|`404`|Número não registado _(se `is_new_user` exigir registo prévio)_|

---

### `POST /auth/biometric/verify`

_(Opcional, complementar ao OTP — RF02)_ Confirma uma operação sensível via biometria local validada no dispositivo.

**Request Body**

```json
{
  "user_id": "usr_8f3a2b",
  "biometric_token": "device_signed_token",
  "action": "RECHARGE_CONFIRM"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": { "verified": true }
}
```

---

### `POST /auth/refresh`

Renova a sessão JWT antes de expirar.

**Request Body**

```json
{ "refresh_token": "eyJhbGciOiJI..." }
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": { "access_token": "eyJhbGciOiJI...", "expires_in": 3600 }
}
```

**Erros**

|Código|Descrição|
|---|---|
|`401`|Sessão expirada — _exige nova autenticação OTP (RN04)_|

---

### `POST /auth/logout`

Invalida a sessão activa.

**Response `200 OK`**

```json
{ "success": true, "data": { "logged_out": true } }
```

---

---

## 3. Módulo `meter` — Gestão de Contadores

_Cobre: RF01, RN09_

### `GET /meters/me`

Lista todos os contadores associados ao utilizador autenticado _(RN09 — acesso restrito aos próprios contadores)_.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "meters": [
      {
        "meter_id": "mtr_a92f1c",
        "serial_number": "CRD-2026-00481",
        "label": "Casa — Beira",
        "location": {
          "latitude": -19.8436,
          "longitude": 34.8389,
          "address": "Bairro Macuti, Beira"
        },
        "status": "ONLINE",
        "credit_kwh": 12.45,
        "relay_state": true,
        "last_recharge_at": "2026-06-20T09:15:00Z"
      }
    ]
  }
}
```

---

### `POST /meters`

Regista um novo contador associado ao utilizador (vinculação inicial).

**Request Body**

```json
{
  "serial_number": "CRD-2026-00481",
  "label": "Casa — Beira",
  "location": {
    "latitude": -19.8436,
    "longitude": 34.8389,
    "address": "Bairro Macuti, Beira"
  }
}
```

**Response `201 Created`**

```json
{
  "success": true,
  "data": { "meter_id": "mtr_a92f1c", "status": "PENDING_ACTIVATION" }
}
```

**Erros**

|Código|Descrição|
|---|---|
|`409`|Número de série já registado por outro utilizador|
|`404`|Número de série não reconhecido pelo sistema EDM|

---

### `GET /meters/{meter_id}`

Detalhe de um contador específico.

**Response `200 OK`** — mesma estrutura do item da listagem acima.

**Erros**

|Código|Descrição|
|---|---|
|`403`|Contador não pertence ao utilizador autenticado (RN09)|
|`404`|Contador não encontrado|

---

### `PATCH /meters/{meter_id}`

Edita metadados do contador (label, localização).

**Request Body**

```json
{ "label": "Casa nova — Munhava" }
```

**Response `200 OK`**

```json
{ "success": true, "data": { "meter_id": "mtr_a92f1c", "updated": true } }
```

---

### `GET /meters/{meter_id}/status`

Estado em tempo real do contador _(RF08 — complementar ao Supabase Realtime para consultas pontuais/polling de fallback)_.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "status": "ONLINE",
    "credit_kwh": 12.45,
    "relay_state": true,
    "last_seen_at": "2026-06-25T14:28:00Z"
  }
}
```

---

---

## 4. Módulo `recharge` — Recargas

_Cobre: RF03, RF16, RN01, RN02, RN10_

### `POST /recharges/initiate`

Inicia o processo de recarga (cria registo `PENDING` antes do pagamento).

**Request Body**

```json
{
  "meter_id": "mtr_a92f1c",
  "amount_mzn": 200.00
}
```

**Response `201 Created`**

```json
{
  "success": true,
  "data": {
    "recharge_id": "rch_7d910e",
    "status": "PENDING",
    "amount_mzn": 200.00,
    "estimated_kwh": 18.2
  }
}
```

**Erros**

|Código|Descrição|
|---|---|
|`403`|Contador não pertence ao utilizador|
|`422`|Valor abaixo do mínimo permitido|

---

### `GET /recharges/{recharge_id}/status`

Consulta o estado de uma recarga em curso.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "recharge_id": "rch_7d910e",
    "status": "CONFIRMED",
    "token": "1234-5678-9012-3456",
    "applied_at": "2026-06-25T14:31:02Z"
  }
}
```

**Estados possíveis:** `PENDING` · `PAYMENT_PROCESSING` · `CONFIRMED` · `MQTT_SENT` · `ACK_RECEIVED` · `FAILED` · `REFUNDED`

---

### `POST /recharges/manual-code`

Aplica um código de recarga CREDELEC obtido por canal externo _(RF16, RN10 — validação de uso único)_.

**Request Body**

```json
{
  "meter_id": "mtr_a92f1c",
  "recharge_code": "9876-5432-1098-7654"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "recharge_id": "rch_3a8b21",
    "status": "MQTT_SENT",
    "credit_kwh": 15.0
  }
}
```

**Erros**

|Código|Descrição|
|---|---|
|`409`|Código já utilizado anteriormente (RN10)|
|`422`|Código inválido ou mal formatado|

---

### `GET /recharges/history`

Histórico de recargas do utilizador _(RF06)_, com filtros.

**Query Params:** `meter_id` (opcional) · `from` · `to` · `page` · `page_size`

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "recharges": [
      {
        "recharge_id": "rch_7d910e",
        "meter_id": "mtr_a92f1c",
        "amount_mzn": 200.00,
        "credit_kwh": 18.2,
        "status": "CONFIRMED",
        "created_at": "2026-06-25T14:30:00Z"
      }
    ],
    "pagination": { "page": 1, "page_size": 20, "total": 47 }
  }
}
```

---

### `GET /recharges/dashboard`

Estatísticas agregadas de consumo _(RF14)_.

**Query Params:** `meter_id` · `period` (`week`/`month`/`year`)

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "total_spent_mzn": 1840.00,
    "total_kwh_purchased": 167.3,
    "average_consumption_kwh_day": 5.4,
    "recharge_count": 9
  }
}
```

---

### `GET /recharges/export`

Exporta histórico em PDF _(RF15)_.

**Query Params:** `meter_id` · `from` · `to`

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "file_url": "https://storage.gezi.mz/exports/rch_export_8f2a.pdf",
    "expires_at": "2026-06-26T14:30:00Z"
  }
}
```

---

---

## 5. Módulo `payment` — Pagamentos (M-Pesa via E2Payments)

_Cobre: RF04, RNF06, RN08_

### `POST /payments/initiate`

Inicia o pagamento associado a uma recarga _(chamado internamente após `/recharges/initiate`, mas exposto para reenvio em caso de timeout)_.

**Request Body**

```json
{
  "recharge_id": "rch_7d910e",
  "msisdn": "+258841234567"
}
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "payment_id": "pay_1c92ab",
    "status": "STK_PUSH_SENT",
    "provider": "M-PESA"
  }
}
```

---

### `GET /payments/{payment_id}`

Consulta o estado de um pagamento.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "payment_id": "pay_1c92ab",
    "status": "SUCCESS",
    "amount_mzn": 200.00,
    "confirmed_at": "2026-06-25T14:30:55Z"
  }
}
```

**Estados possíveis:** `STK_PUSH_SENT` · `AWAITING_PIN` · `SUCCESS` · `FAILED` · `CANCELLED` · `TIMEOUT`

---

### `POST /payments/callback`  _(Webhook — sem autenticação JWT)_

Recebido do **E2Payments**, confirma o resultado do pagamento. Protegido por validação de assinatura HMAC do gateway, não por JWT de utilizador.

**Request Body** _(formato do E2Payments)_

```json
{
  "reference": "pay_1c92ab",
  "status": "SUCCESS",
  "transaction_id": "MPESA987654321",
  "amount": 200.00,
  "signature": "a1b2c3..."
}
```

**Response `200 OK`**

```json
{ "success": true, "data": { "processed": true } }
```

> Este endpoint despoleta internamente: gravação `INSERT transaction` (RN03, imutável) → geração do token STS → publicação MQTT `APPLY_CREDITS`.

**Erros**

|Código|Descrição|
|---|---|
|`401`|Assinatura HMAC inválida — possível tentativa de falsificação|
|`409`|Callback duplicado para a mesma referência (idempotência)|

---

---

## 6. Módulo `iot` — Comunicação com Dispositivos ESP32

_Cobre: RF05, RF11, RF12, RN05, RN06, RN07, RNF08_

### `POST /iot/commands/apply-credits` _(interno)_

Publica o comando MQTT `APPLY_CREDITS` para o ESP32. Não chamado directamente pelo cliente mobile — invocado internamente pelo módulo `recharge`/`payment`.

**Payload MQTT publicado** _(tópico: `credelec/meter/{meter_id}/cmd`, QoS 1 — RNF08)_

```json
{
  "command": "APPLY_CREDITS",
  "token": "1234-5678-9012-3456",
  "kwh": 18.2,
  "hmac": "f9e8d7...",
  "issued_at": "2026-06-25T14:31:00Z"
}
```

---

### `POST /iot/commands/cut-supply` _(interno/admin)_

Publica comando de corte manual _(uso administrativo — RF10)_.

**Tópico MQTT:** `credelec/meter/{meter_id}/cmd` · QoS 1

```json
{ "command": "CUT_SUPPLY", "reason": "MANUAL_ADMIN", "hmac": "..." }
```

---

### `POST /iot/commands/restore-supply` _(interno/admin)_

Publica comando de religação manual.

**Tópico MQTT:** `credelec/meter/{meter_id}/cmd` · QoS 1

```json
{ "command": "RESTORE_SUPPLY", "hmac": "..." }
```

---

### `POST /iot/telemetry` _(Subscrito via MQTT, não HTTP directo)_

> Nota: este "endpoint" representa o **handler interno do subscriber MQTT**, não uma rota HTTP pública. Documentado aqui por completude do contrato de dados.

**Tópico subscrito:** `credelec/meter/{meter_id}/telemetry` · QoS 0

```json
{
  "voltage": 220.4,
  "current": 2.31,
  "power_w": 509.0,
  "frequency": 50.0,
  "energy_kwh": 1.25,
  "credit_kwh": 12.45,
  "relay_state": true,
  "timestamp": "2026-06-25T14:32:00Z"
}
```

Processamento: grava em `meter_telemetry` (Supabase) → dispara Realtime → se `credit_kwh < threshold`, gera alerta (RF07).

---

### `POST /iot/ack` _(Subscrito via MQTT)_

**Tópico subscrito:** `credelec/meter/{meter_id}/ack`

```json
{
  "command_id": "cmd_5f8a2c",
  "status": "ACK",
  "applied_kwh": 18.2,
  "timestamp": "2026-06-25T14:31:05Z"
}
```

Processamento: actualiza `command_log.status = ACK` (RF11).

---

### `GET /iot/commands/{command_id}/log`

Consulta o registo auditável de um comando _(RF11)_.

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "command_id": "cmd_5f8a2c",
    "command": "APPLY_CREDITS",
    "meter_id": "mtr_a92f1c",
    "status": "ACK",
    "attempts": 1,
    "sent_at": "2026-06-25T14:31:00Z",
    "acked_at": "2026-06-25T14:31:05Z"
  }
}
```

**Lógica de retry (RN06):** se não houver ACK em 10s, o serviço reenvia até 3 tentativas com backoff exponencial (2s, 4s, 8s). Após a 3ª falha → `status: FAILED` + reembolso automático via `/payments/{id}/refund`.

---

### `POST /iot/fallback/validate-token` _(RF12 — fallback offline)_

Valida localmente (no próprio ESP32, via firmware) um token inserido no teclado decimal **sem ligação à internet**. Este endpoint existe no backend apenas para **pré-gerar e sincronizar a lista de tokens válidos** que o ESP32 guarda em NVS.

**Request Body**

```json
{ "meter_id": "mtr_a92f1c", "batch_size": 10 }
```

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "tokens": [
      { "token": "111122223333", "kwh": 5.0, "expires_at": "2026-07-25T00:00:00Z" }
    ]
  }
}
```

> O ESP32 guarda este lote localmente (RN07) e valida offline comparando o token inserido via keypad com a lista em NVS, decrementando o lote após uso.

---

---

## 7. Módulo `admin` — Painel Administrativo EDM

_Cobre: RF09, RF10_

### `GET /admin/meters`

Lista todos os contadores do sistema _(role: `ADMIN_EDM`)_.

**Query Params:** `status` · `page` · `page_size`

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "meters": [ { "meter_id": "mtr_a92f1c", "status": "ONLINE", "credit_kwh": 12.45 } ],
    "pagination": { "page": 1, "total": 1842 }
  }
}
```

---

### `GET /admin/transactions`

Lista transacções financeiras do sistema, com filtros.

**Query Params:** `status` · `from` · `to` · `meter_id`

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "transactions": [
      { "recharge_id": "rch_7d910e", "amount_mzn": 200.00, "status": "CONFIRMED" }
    ]
  }
}
```

---

### `GET /admin/alerts`

Lista alertas operacionais (falhas MQTT, saldo crítico, dispositivos offline).

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "alerts": [
      { "type": "DEVICE_OFFLINE", "meter_id": "mtr_b21f9a", "since": "2026-06-25T10:00:00Z" }
    ]
  }
}
```

---

### `GET /admin/metrics`

Métricas operacionais agregadas (RF10).

**Response `200 OK`**

```json
{
  "success": true,
  "data": {
    "total_meters": 1842,
    "online_now": 1791,
    "total_revenue_mzn_today": 48200.00,
    "failed_commands_today": 3,
    "avg_command_latency_ms": 850
  }
}
```

---

---

## 8. Documentos e Storage

_Cobre: RF13_

### `POST /documents/upload`

Carrega comprovativo de pagamento ou relatório.

**Request:** `multipart/form-data` — campo `file` + `recharge_id` (opcional)

**Response `201 Created`**

```json
{
  "success": true,
  "data": { "document_id": "doc_2f8a1c", "file_url": "https://storage.gezi.mz/docs/..." }
}
```

---

### `GET /documents/{document_id}`

Devolve metadados e URL de acesso ao documento.

**Response `200 OK`**

```json
{
  "success": true,
  "data": { "document_id": "doc_2f8a1c", "type": "PAYMENT_RECEIPT", "file_url": "..." }
}
```

---

---

## 9. Tabela-Resumo de Endpoints

| Método | Rota                           | Módulo    | RF/RN      | Auth  |
| ------ | ------------------------------ | --------- | ---------- | ----- |
| POST   | `/auth/otp/request`            | auth      | RF02       | Não   |
| POST   | `/auth/otp/verify`             | auth      | RF02       | Não   |
| POST   | `/auth/biometric/verify`       | auth      | RF02       | Sim   |
| POST   | `/auth/refresh`                | auth      | RN04       | Sim   |
| POST   | `/auth/logout`                 | auth      | RN04       | Sim   |
| GET    | `/meters/me`                   | meter     | RF01, RN09 | Sim   |
| POST   | `/meters`                      | meter     | RF01       | Sim   |
| GET    | `/meters/{id}`                 | meter     | RF01, RN09 | Sim   |
| PATCH  | `/meters/{id}`                 | meter     | RF01       | Sim   |
| GET    | `/meters/{id}/status`          | meter     | RF08       | Sim   |
| POST   | `/recharges/initiate`          | recharge  | RF03, RN01 | Sim   |
| GET    | `/recharges/{id}/status`       | recharge  | RF03       | Sim   |
| POST   | `/recharges/manual-code`       | recharge  | RF16, RN10 | Sim   |
| GET    | `/recharges/history`           | recharge  | RF06       | Sim   |
| GET    | `/recharges/dashboard`         | recharge  | RF14       | Sim   |
| GET    | `/recharges/export`            | recharge  | RF15       | Sim   |
| POST   | `/payments/initiate`           | payment   | RF04       | Sim   |
| GET    | `/payments/{id}`               | payment   | RF04       | Sim   |
| POST   | `/payments/callback`           | payment   | RF04       | HMAC  |
| GET    | `/iot/commands/{id}/log`       | iot       | RF11       | Sim   |
| POST   | `/iot/fallback/validate-token` | iot       | RF12       | Sim   |
| GET    | `/admin/meters`                | admin     | RF10       | Admin |
| GET    | `/admin/transactions`          | admin     | RF10       | Admin |
| GET    | `/admin/alerts`                | admin     | RF10       | Admin |
| GET    | `/admin/metrics`               | admin     | RF10       | Admin |
| POST   | `/documents/upload`            | documents | RF13       | Sim   |
| GET    | `/documents/{id}`              | documents | RF13       | Sim   |

---

## 10. Tópicos MQTT (Resumo)

|Tópico|Direcção|QoS|Payload|
|---|---|---|---|
|`credelec/meter/{id}/cmd`|Servidor → ESP32|1|APPLY_CREDITS / CUT_SUPPLY / RESTORE_SUPPLY|
|`credelec/meter/{id}/telemetry`|ESP32 → Servidor|0|voltage, current, power, energy, credit_kwh|
|`credelec/meter/{id}/ack`|ESP32 → Servidor|1|command_id, status, applied_kwh|

**Autenticação MQTT (RN05):** certificado X.509 único por dispositivo, validado pelo broker antes de qualquer pub/sub no namespace `credelec/*`.

---

## 11. Considerações de Segurança Transversais

- **TLS 1.3** obrigatório em todas as ligações HTTPS e MQTT (RNF01)
- **HMAC-SHA256** assina todos os comandos MQTT críticos, validado pelo firmware antes de aplicar qualquer crédito
- **Rate limiting** em `/auth/otp/request` para mitigar abuso de SMS
- **Idempotência** obrigatória em `/payments/callback` (chave: `reference`) para evitar duplicação de créditos em caso de reenvio do webhook
- **RLS (Row Level Security)** no Supabase garante RN09 a nível de base de dados, não apenas a nível de aplicação