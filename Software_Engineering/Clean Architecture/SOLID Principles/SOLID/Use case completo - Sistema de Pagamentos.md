**Cenário:** aplicação mobile que precisa processar pagamentos via múltiplos provedores (M-Pesa, cartão de crédito e PayPal).

Aplicando todos os princípios:

> ***Open-Close & Dependency Inversion Principles (OCP & DIP)):***

```dart title:payment_gateway.dart
// ABSTRACAO (OCP, DIP)
abstract class PaymentGateway {
	Future<PaymentResult> processPayment(double amount, PaymentDetails details); // evitar usar tipos primitivos para moedas ($amount), mas para caso de compreensao podemos seguir
}
```

> ***Single Responsibility Principle (SRP):***

```dart title:mpesa_gateway_interface.dart
// Cada gateway so processa pagamento da sua forma
class MpesaGateway implements PaymentGateway {
	final MpesaAPI _api;
	
	MpesaGateway(this._api); // DIP - dependecia de abstracao
	
	@override Future<PaymentResult> processPayment(double amount, PaymentDetails details) async {
		// logica especifica de mpesa
		return await _api.pay(amount, details.phoneNumber);
	}
}
```

```dart title:creditcard_gateway_interface.dart
class CreditCard implements PaymentGateway {
	final CardProcessor _processor;
	
	CreditCardGateway(this._processor); // DIP
	
	@override
	Future<PaymentResult> processPayment(double amount, PaymentDetails details) async {
		// logica de credit card
		return await _processor.charge(amount, details.cardNumber);
	}
}
```

> ***Interface Segregation Principle (ISP):***

```dart title:payment_logger.dart
// ISP - Interface segregada para logging
abstract class PaymentLogger {
	void logTransaction(String transactionId, double amount);
}
```

```dart title:payment_validator.dart
// ISP - Interface segregada para validacao
abstract class PaymentValidator {
	bool validateAmount(double amount);
	bool validateDetails(PaymentDetails details);	
}
```

> ***SRP - classe focada em orquestração:***
```dart title:payment_service.dart
class PaymentService {
	final PaymentGateway gateway;
	final PaymentLogger logger;
	final PaymentValidator validator;
	
	PaymentService(this.gateway, this.logger, this.validator); // DIP
	
	Future<PaymentResult> makePayment(double amount, PaymentDetails details) async {
		// Valida
		if (!validator.validateAmount(amount) || !validator.validateDetails(details)) {
			throw InvalidPaymentException();
		}
		
		// Processa
		final result = await gateway.processPayment(amount, details);
		
		// Logging
		logger.logTransaction(result.transactionId, amount);
		
		return result;
	}
}
```

> ***Liskov Substitution Principle (LSP) - todas implementações respeitam o contrato:***

```dart title:test_gateway_interface.dart
class FakePaymentGateway implements PaymentGateway {
	@override
	Future<PaymentResult> processPayment(double amount, PaymentDetails details) async {
		return PaymentResult(transactionId: 'FAKE-123', success: true);
	}
}
```

> ***OCP - podemos adicionar novo gateway sem modificar código existente:***

```dart title:paypal_gateway_interface.dart
class PayPalGateway implements PaymentGateway {
	final PayPalAPI _api;
	
	PayPalGateway(this._api);
	
	@override
	Future<PaymentResult> processPayment(double amount, PaymentDetails details) async {
		return await _api.checkout(amount, details.email);
	}
}
```

### **CONCLUSÃO**

**Clean Architecture** do Uncle Bob organiza código em camadas (veremos depois):

- **Entities** (regras de negócio)
- **Use Cases** (casos de uso da aplicação)
- **Interface Adapters** (controllers, presenters)
- **Frameworks & Drivers** (UI, DB, APIs externas)

**SOLID torna isso possível:**

- **SRP**: Cada camada tem uma responsabilidade
- **OCP**: Podemos estender sem modificar camadas internas
- **LSP**: Camadas externas podem substituir implementações
- **ISP**: Interfaces entre camadas são focadas
- **DIP**: Camadas internas não dependem de externas (dependem de abstrações)

**Regra de Dependência:** Dependências apontam SEMPRE para dentro (para as camadas mais estáveis).

```
┌─────────────────────────────────────────┐
│         Frameworks & Drivers            │  ← Mais instável
│    (Flutter, Firebase, APIs)            │
├─────────────────────────────────────────┤
│      Interface Adapters                 │
│   (Controllers, Presenters, Gateways)   │
├─────────────────────────────────────────┤
│          Use Cases                      │
│    (Application Business Rules)         │
├─────────────────────────────────────────┤
│          Entities                       │  ← Mais estável
│    (Enterprise Business Rules)          │
└─────────────────────────────────────────┘

     Dependências apontam para DENTRO →
```

[[Princípios SOLID explicados com use cases]]