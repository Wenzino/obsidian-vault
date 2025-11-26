*"Objetos de uma classe derivada devem poder substituir objetos da classe base sem quebrar a aplicação"*

**Explicação:** Se `B` é subtipo de `A`, então posso substituir `A` por `B` sem alterar o comportamento esperado do programa

**Use case - Sistema de notificações:**

```dart title:notification_service.dart
// VIOLACAO LSP
abstract class NotificationService {
	Future <void> send(String recipient, String message);
}
```

```dart title:email_service.dart
class EmailService implements NotificationService {
	@override 
	Future<void> send(String recipient, String message) async {
		// Funciona normalmente
		await sendEmail(recipient, message);
	}
}
```

```dart title:sms_service.dart
class SMSService implements NotificationService {
	@override
	Future <void> send(String recipient, String message) async {
		// VIOLA LSP: lanca excecao para mensagens longas!
		if (message.length > 160) {
			throw Exception('SMS muito longa!');
		}
		await sendSMS(recipient, message);
	}
}
```

```dart title:notify_users_client.dart
// Codigo cliente - falha quando substitui por SMSService
void notifyUsers(NotificationService service, List<User> users) {
	for (var user in users) {
		// Isto funciona com Email mas FALHA com SMS se mensagem > 160
		service.send(user.contact, 'Mensagem longa...');
	}
}
```

**SOLUÇÃO:**

```dart title:notification_service.dart
// Respeita LSP
abstract class NotificationService {
	Future<void> send(String recipient, String message);
	int get maxMessageLength; //contrato claro sobre limitacoes
}
```

```dart title:email_service.dart
class EmailService implements NotificationService {
	@override
	int get maxMessageLength => 10000;
	@override
	Future<void> send(String recipient, String message) async {
		if (message.length > maxMessageLength) {
			throw Exception('Mensagem Excede o limite');
		}
		
		await sendEmail(recipient, message);
	}
}
```

```dart title:sms_service.dart
class SMSService implements NotificationService {
	@override
	int get maxMessageLength => 160;
	@override
	Future<void> send(String recipient, String message) async {
		if (message.length > maxMessageLength) {
			throw Exception('Mensagem excede limite');
		}
		
		await sendSMS(recipient, message);
	}
}
```

```dart title:notify_users_client.dart
// codigo cliente que funciona com qualquer IMPLEMENTACAO
void notifyUsers(NotificationService service, List<User> users) {
	for (var user in users) {
		String message = 'Mensagem original...';
		
		//Respeita o contrato de cada servico;
		if (message.length > service.maxMessageLength) {
			message = message.substring(0, service.maxMessageLength);
		}
		
		service.send(user.contact, message);
	}
}
```

**Benefícios:**
- Comportamento previsível em todos os subtipos 
- Sem surpresas ao substituir implementações
- Contrato claro e respeitado