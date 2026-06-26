> Resumo estruturado e orientado à implementação da API REST do **E2Payments**, baseado na documentação oficial (`https://e2payments.explicador.co.mz/docs/api`), no SDK PHP oficial e em tutoriais publicados pela Explicador. Cobre Mpesa e eMola — os dois métodos de carteira móvel actualmente documentados publicamente.

---

## 1. Visão geral

O **E2Payments** é uma plataforma da **Explicador Inc.** que funciona como camada intermediária (gateway) entre o teu sistema e as APIs das instituições financeiras (Mpesa/Vodacom, eMola/Movitel, BIM, Standard Bank). Permite processar pagamentos via carteira móvel, Visa/Mastercard, sem que precises de lidar directamente com a complexidade de cada API bancária.

Pontos-chave:

- Arquitectura **REST**, todas as chamadas são **HTTP POST** (mesmo para "leitura" de dados — não há GET).
- Autenticação via **OAuth2 client_credentials** (`client_id` + `client_secret` → `access_token`).
- O conceito central é a **carteira (wallet)** — cada transação está sempre associada a um `wallet_id`.
- **A Explicador não é um banco.** A carteira é apenas um instrumento de gestão/visualização; o dinheiro real permanece sempre na tua conta Mpesa/banco.

**URL base:** `https://e2payments.explicador.co.mz` **Prefixo da API:** `/v1/`

---

## 2. Conceitos fundamentais

### 2.1 Carteira (Wallet)

Para teres uma carteira no E2Payments precisas de:

1. Ter uma **conta de negócio (Business Account)** junto do Mpesa, BIM ou Standard Bank — não é uma conta pessoal/singular, exige empresa registada com contrato com o banco.
2. Obter as credenciais de API dessa instituição (no caso do Mpesa, via [portal de desenvolvedor Mpesa](https://developer.mpesa.vm.co.mz/)).
3. Registar essa carteira no painel administrativo do E2Payments → obténs um `wallet_id` (aparece prefixado com `#` no painel; usa-se **sem** o `#` nas chamadas).

### 2.2 Transação

Uma transação é a troca de dinheiro entre contas Mpesa↔Mpesa ou conta bancária↔conta bancária (na v1 da API não há transferência cross-method, ex.: Mpesa→Banco).

|Tipo|Significado|Exemplo|
|---|---|---|
|**C2B**|_Customer to Business_ — dinheiro sai do cliente para o negócio|Cliente paga uma compra online|
|**B2C**|_Business to Customer_ — dinheiro sai do negócio para o cliente|Reembolso, pagamento a colaborador|
|**B2B**|_Business to Business_ — dinheiro sai de um negócio para outro|Pagamento a fornecedor|

>  A documentação pública detalha em profundidade apenas o fluxo **C2B** (Mpesa e eMola). B2C/B2B existem na plataforma mas o detalhe de payload não está completamente publicado — ver secção 6.4.

---

## 3. Autenticação

### 3.1 Obter credenciais

1. Inicia sessão numa conta Explicador (email confirmado).
2. Acede ao painel: `https://e2payments.explicador.co.mz/admin/credentials`.
3. Cria uma credencial preenchendo:
    - **Name** — identificador da credencial (recomenda-se nomear por aplicativo).
    - **Redirect URL** — domínio/URL do teu aplicativo (ex.: `https://app.kodeza.co.mz`). **Requisições vindas de um domínio fora desta lista são bloqueadas pelo servidor.**
4. O sistema gera:

```json
{
  "client_id": "91fdae03-29a0-496c-9451-fb1e7dd2adffdf",
  "client_secret": "T8sHdLfujgjZBp8aAf0Gsfu3kJgDzmNRUGEdN0sdfdsf"
}
```

> `client_secret` só deve ser usado para gerar o token (secção seguinte). Em qualquer outra chamada **não envies o client_secret**, apenas o `client_id`.

### 3.2 Gerar o Access Token

```
POST https://e2payments.explicador.co.mz/oauth/token
Content-Type: application/json
```

```javascript
const axios = require('axios');

const { data } = await axios.post(
  'https://e2payments.explicador.co.mz/oauth/token',
  {
    grant_type: 'client_credentials',
    client_id: process.env.E2P_CLIENT_ID,
    client_secret: process.env.E2P_CLIENT_SECRET,
  }
);
```

**Resposta:**

```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIm...",
  "expires_in": 31536000,
  "token_type": "Bearer"
}
```

`expires_in` está em segundos — **31536000 = 1 ano**. Ou seja, o token é de muito longa duração.

>  **Nota de segurança:** um token válido por 1 ano implica que, se for comprometido, o atacante tem uma janela enorme de exploração. Em produção, recomenda-se gerar o token apenas no **servidor** (nunca no browser/cliente), guardá-lo de forma cifrada (variável de ambiente / secrets manager) e implementar rotação manual periódica das credenciais no painel, independentemente da validade declarada do token.

### 3.3 Onde guardar o token

A documentação oficial sugere armazenar o token num **cookie do browser** (cenário client-side / SPA simples):

```javascript
async function storeTokenInBrowser(responseData) {
  const token = responseData.token_type + ' ' + responseData.access_token;
  const expires = addDaysToToken(10); // validade local de 10 dias
  document.cookie = `e2payment_token=${token || ''}${expires}; path=/`;
}

function addDaysToToken(totalDays) {
  Date.prototype.addDays = function (days) {
    const date = new Date(this.valueOf());
    date.setDate(date.getDate() + days);
    return date;
  };
  return new Date().addDays(totalDays);
}
```

> **Recomendação para projectos Kodeza (NestJS/backend):** não sigas este padrão client-side em produção. Gera e guarda o token no **backend** (ex.: Redis com TTL, ou tabela `integration_tokens`), e exponhas apenas os teus próprios endpoints internos ao frontend. O cookie no browser expõe o Bearer token a qualquer pessoa que inspeccione o site — a documentação assume que a protecção fica inteiramente a cargo da validação de `Redirect URL` no servidor do E2Payments, o que é uma mitigação fraca para uma SPA pública.

---

## 4. Composição do Header (igual em todas as chamadas, exceto `/oauth/token`)

```json
{
  "Authorization": "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIm...",
  "Accept": "application/json",
  "Content-Type": "application/json"
}
```

- Falta de `Authorization` válido → **401 Unauthenticated**.
- Todas as requisições devem ser **POST**, mesmo as de "consulta" — o `client_id` viaja sempre no corpo (body), nunca apenas na query string.

---

## 5. Gestão de Carteiras (Wallets)

|Acção|Método|Endpoint|
|---|---|---|
|Listar todas as carteiras (Mpesa)|`POST`|`/v1/wallets/mpesa/get/all`|
|Listar todas as carteiras (eMola)|`POST`|`/v1/wallets/emola/get/all`|
|Detalhes de uma carteira (Mpesa)|`POST`|`/v1/wallets/mpesa/get/{walletId}`|

**Payload (igual para todas):**

```json
{ "client_id": "91fdae03-29a0-496c-9451-fb1e7dd2adffdf" }
```

```javascript
const ENDPOINT_URL = 'https://e2payments.explicador.co.mz/v1/wallets/mpesa/get/all';

await axios.post(
  ENDPOINT_URL,
  { client_id: process.env.E2P_CLIENT_ID },
  { headers: header }
);
```

---

## 6. Transações

### 6.1 C2B — Mpesa

```
POST /v1/c2b/mpesa-payment/{wallet_id}
```

**Payload:**

```json
{
  "client_id": "91fdae03-29a0-496c-9451-fb1e7dd2adffdf",
  "amount": "30",
  "phone": "848512345",
  "reference": "PrimeiroPagamento"
}
```

Regras do payload:

- `amount` — valor da transacção (string ou número, conforme exemplos da doc).
- `phone` — **9 dígitos**, sem código de país (ex.: `84xxxxxxx` / `85xxxxxxx`).
- `reference` — string **sem espaços**; é o texto que aparece no popup/SMS de confirmação Mpesa no telemóvel do cliente.

```javascript
const wallet_id = 123456;
const ENDPOINT_URL = `https://e2payments.explicador.co.mz/v1/c2b/mpesa-payment/${wallet_id}`;

const payload = {
  client_id: process.env.E2P_CLIENT_ID,
  amount: '30',
  phone: '848512345',
  reference: 'PrimeiroPagamento',
};

await axios.post(ENDPOINT_URL, payload, { headers: header });
```

Fluxo esperado: o telemóvel associado ao `phone` recebe um pop-up Mpesa solicitando o PIN para confirmar o pagamento de `amount`. A transação só fica concluída quando o cliente introduz o PIN no telemóvel.

### 6.2 C2B — eMola

```
POST /v1/c2b/emola-payment/{wallet_id}
```

Payload **idêntico** ao do Mpesa (mesmos campos: `client_id`, `amount`, `phone`, `reference`), apenas o endpoint muda. O intervalo documentado para `amount` é de **1 a 1.250.000 MT**.

```javascript
const ENDPOINT_URL = `https://e2payments.explicador.co.mz/v1/c2b/emola-payment/${wallet_id}`;
```

> Recomenda-se testar o fluxo eMola em sandbox antes de produção — a documentação pública não detalha explicitamente se o passo de confirmação no telemóvel do cliente segue o mesmo padrão de pop-up do Mpesa STK Push, devendo este comportamento ser validado empiricamente com a tua conta de negócio eMola.

### 6.3 Resposta de uma transacção (HTTP Status Codes)

|Código|Significado|Causa típica|
|---|---|---|
|`200`|OK — requisição executada e registada|—|
|`201`|Added — transacção criada e registada|—|
|`400`|Bad Request|Problema na carteira, origem ou validação Mpesa/eMola|
|`401`|Unauthenticated|`client_id`/`client_secret` inválidos ou token ausente/expirado|
|`403`|Forbidden|`wallet_id` inexistente ou não pertence ao utilizador (se for de uma organização, é preciso ter sido convidado)|
|`500`|Server error|Erro do lado da Explicador — reportar à equipa|

### 6.4 B2C e B2B (padrão esperado — confirmar no painel)

A documentação pública foca-se no C2B. Seguindo a convenção REST usada pelo C2B (`/v1/{tipo}/{metodo}-payment/{wallet_id}`), o padrão para os outros tipos seria:

```
POST /v1/b2c/mpesa-payment/{wallet_id}
POST /v1/b2b/mpesa-payment/{wallet_id}
POST /v1/b2c/emola-payment/{wallet_id}
POST /v1/b2b/emola-payment/{wallet_id}
```

>  **Não assumir sem validar.** Estes endpoints não estão exemplificados com payload completo na documentação pública consultada. Antes de implementar B2C/B2B em produção: (1) confirma os endpoints exactos e os campos obrigatórios directamente no painel/suporte do E2Payments, e (2) testa primeiro com valores baixos numa carteira de sandbox, se disponível.

---

## 7. Histórico de Pagamentos

|Acção|Endpoint|
|---|---|
|Todos os pagamentos (Mpesa)|`POST /v1/payments/mpesa/get/all`|
|Todos os pagamentos (eMola)|`POST /v1/payments/emola/get/all`|
|Paginado (Mpesa)|`POST /v1/payments/mpesa/get/all/paginate/{qtdDePagamentos}`|
|Paginado (eMola)|`POST /v1/payments/emola/get/all/paginate/{qtdDePagamentos}`|

```javascript
const qtdDePagamentos = 10;
const ENDPOINT_URL =
  `https://e2payments.explicador.co.mz/v1/payments/mpesa/get/all/paginate/${qtdDePagamentos}`;

const { data } = await axios.post(
  ENDPOINT_URL,
  { client_id: process.env.E2P_CLIENT_ID },
  { headers: header }
);

console.log(data);              // lista de pagamentos
console.log(data.next_page_url); // URL para a próxima página
```

>  Usa a versão **paginada** sempre que possível em produção — a versão "get/all" sem paginação pode tornar-se lenta/pesada à medida que o histórico cresce para milhares de registos (alerta explícito na documentação oficial).

---

## 8. Boas práticas de segurança e implementação

1. **Nunca exponhas `client_secret` no frontend.** Só é necessário uma vez, na troca por token, e essa troca deve ser sempre feita pelo backend.
2. **Centraliza a geração/cache do token** no backend (Redis/DB com TTL inferior ao `expires_in` real, por segurança), evitando gerar token a cada requisição.
3. **Valida sempre o `Redirect URL`** configurado no painel — é a única camada de protecção contra uso indevido de um token exposto.
4. **Idempotência:** a API não documenta um mecanismo nativo de idempotency key. Implementa do teu lado um controlo (ex.: chave única por `reference` + `wallet_id` + timestamp) para evitar duplicar cobranças em caso de retry de rede.
5. **Reconciliação:** como não há (publicamente documentado) endpoint de _consulta de estado de transacção individual_ nem _webhook/callback_, o padrão seguro é fazer **polling periódico** do histórico de pagamentos (`/v1/payments/.../get/all/paginate/{n}`) e casar pelos campos `reference`/`amount`/`phone`/timestamp para confirmar conclusão.
6. **Trata todos os códigos de erro** (400/401/403/500) explicitamente — não assumas sucesso apenas por ausência de exceção de rede.
7. **Regista todas as chamadas (request/response) em log próprio**, incluindo timestamps — útil tanto para suporte com a Explicador como para auditoria interna.

---

## 9. Exemplo de implementação completa (NestJS)

Exemplo de um serviço de integração reutilizável, adequado ao stack que a Kodeza já usa noutros projectos (NestJS + Clean Architecture):

```typescript
// e2payments.config.ts
export interface E2PaymentsConfig {
  baseUrl: string;       // https://e2payments.explicador.co.mz
  clientId: string;
  clientSecret: string;
}
```

```typescript
// e2payments.service.ts
import { Injectable, Logger, BadGatewayException } from '@nestjs/common';
import axios, { AxiosInstance } from 'axios';

interface TokenResponse {
  access_token: string;
  expires_in: number;
  token_type: string;
}

@Injectable()
export class E2PaymentsService {
  private readonly logger = new Logger(E2PaymentsService.name);
  private readonly http: AxiosInstance;
  private cachedToken?: { value: string; expiresAt: number };

  constructor(private readonly config: E2PaymentsConfig) {
    this.http = axios.create({ baseURL: config.baseUrl, timeout: 15000 });
  }

  private async getToken(): Promise<string> {
    const now = Date.now();
    if (this.cachedToken && this.cachedToken.expiresAt > now) {
      return this.cachedToken.value;
    }

    const { data } = await this.http.post<TokenResponse>('/oauth/token', {
      grant_type: 'client_credentials',
      client_id: this.config.clientId,
      client_secret: this.config.clientSecret,
    });

    this.cachedToken = {
      value: `${data.token_type} ${data.access_token}`,
      // cache conservador: renova a cada 24h mesmo que o token dure 1 ano
      expiresAt: now + 24 * 60 * 60 * 1000,
    };

    return this.cachedToken.value;
  }

  private async headers() {
    return {
      Authorization: await this.getToken(),
      Accept: 'application/json',
      'Content-Type': 'application/json',
    };
  }

  async c2bMpesa(params: {
    walletId: string;
    amount: number;
    phone: string;
    reference: string;
  }) {
    const { walletId, amount, phone, reference } = params;

    try {
      const { data } = await this.http.post(
        `/v1/c2b/mpesa-payment/${walletId}`,
        {
          client_id: this.config.clientId,
          amount: String(amount),
          phone,
          reference,
        },
        { headers: await this.headers() },
      );
      return data;
    } catch (err) {
      this.logger.error('Falha na transação C2B Mpesa', err);
      throw new BadGatewayException('Não foi possível processar o pagamento Mpesa.');
    }
  }

  async c2bEmola(params: {
    walletId: string;
    amount: number;
    phone: string;
    reference: string;
  }) {
    const { walletId, amount, phone, reference } = params;

    const { data } = await this.http.post(
      `/v1/c2b/emola-payment/${walletId}`,
      {
        client_id: this.config.clientId,
        amount: String(amount),
        phone,
        reference,
      },
      { headers: await this.headers() },
    );
    return data;
  }

  async listPayments(method: 'mpesa' | 'emola', limit?: number) {
    const path = limit
      ? `/v1/payments/${method}/get/all/paginate/${limit}`
      : `/v1/payments/${method}/get/all`;

    const { data } = await this.http.post(
      path,
      { client_id: this.config.clientId },
      { headers: await this.headers() },
    );
    return data;
  }

  async listWallets(method: 'mpesa' | 'emola') {
    const { data } = await this.http.post(
      `/v1/wallets/${method}/get/all`,
      { client_id: this.config.clientId },
      { headers: await this.headers() },
    );
    return data;
  }
}
```

---

## 10. Resumo de endpoints (referência rápida)

|Recurso|Método|Endpoint|
|---|---|---|
|Token de acesso|`POST`|`/oauth/token`|
|Listar carteiras Mpesa|`POST`|`/v1/wallets/mpesa/get/all`|
|Listar carteiras eMola|`POST`|`/v1/wallets/emola/get/all`|
|Detalhe de carteira Mpesa|`POST`|`/v1/wallets/mpesa/get/{walletId}`|
|C2B Mpesa|`POST`|`/v1/c2b/mpesa-payment/{wallet_id}`|
|C2B eMola|`POST`|`/v1/c2b/emola-payment/{wallet_id}`|
|B2C Mpesa _(verificar)_|`POST`|`/v1/b2c/mpesa-payment/{wallet_id}`|
|B2B Mpesa _(verificar)_|`POST`|`/v1/b2b/mpesa-payment/{wallet_id}`|
|Histórico Mpesa (todos)|`POST`|`/v1/payments/mpesa/get/all`|
|Histórico Mpesa (paginado)|`POST`|`/v1/payments/mpesa/get/all/paginate/{qtd}`|
|Histórico eMola (todos)|`POST`|`/v1/payments/emola/get/all`|
|Histórico eMola (paginado)|`POST`|`/v1/payments/emola/get/all/paginate/{qtd}`|

---

## 11. Lacunas observadas (relevante para decisões futuras da Kodeza)

Durante a análise da documentação pública, identificámos pontos que **não estão cobertos** ou estão pouco detalhados:

- **Sem endpoint de consulta de estado de transacção individual** (ex.: "dá-me o status do `transaction_id X`") — só há listagem/histórico geral.
- **Sem webhooks/callbacks documentados** para notificação assíncrona de confirmação de pagamento — força _polling_.
- **Sem mecanismo de idempotency key** explícito no payload.
- **Sem endpoint de reembolso (refund)** documentado.
- **B2C/B2B** com payload pouco detalhado publicamente.
- **Token de longa duração (1 ano)** sem refresh token nem revogação documentada granularmente.
- **Sem SDK oficial em Node.js/TypeScript** (existe apenas SDK PHP oficial; o resto é comunidade).

Estes pontos são exactamente o tipo de lacuna que justificaria um gateway próprio mais robusto — cobrirei isso na segunda parte, conforme pediste.

---

### Referências consultadas

- Documentação oficial: `https://e2payments.explicador.co.mz/docs/api`
- SDK PHP oficial: `https://github.com/Explicador/e2Payments-php-sdk`
- Tutorial oficial eMola: `https://explicador.co.mz/posts/tutorial-de-integracao-do-metodo-de-pagamento-emola-com-a-api-de-e2payments`