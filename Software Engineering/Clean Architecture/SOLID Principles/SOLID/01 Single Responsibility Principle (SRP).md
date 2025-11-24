Segundo o Robert Martin AKA Uncle Bob no seu livro [[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)]:
> *"Um módulo[^1] deve ser responsável por um, e apenas um, ator. E uma classe deve ter apenas uma razão para mudar."* 

Cada classe/módulo deve ter apenas uma responsabilidade ou fazer apenas uma coisa. Se tens múltiplas razões para modificar uma classe, ela tem múltiplas responsabilidades.

**Use Case - Sistema de notificações:**

``` dart title:notification_manager.dart
// Violacao SRP - multiplas responsabilidades
class NotificationManager {
	// responsabilidade 1: enviar notificacao
	Future<void> sendEmail(String to , String message) async {
		await ExternalEmailAPI.send(to, message);
	}
	
	// Responsabilidade 2: Formatar mensagem
	String formatMessage(String template, Map<String, String> data) {
		return template.replaceAll('{{name}}', data['name'] ?? '');
	}
	
	//Responsabilidade 3: registrar logs
	void logNotification(String recipient, String status) {
		File('logs.txt').writeAsStringSync('$recipient - $status\n');
	}
	
	// Responsabilidade 4: Validar destinatario
	bool isValidEmail(String email) {
		return email.contains('@');
	}
}
```

**Razões para mudar esta classe:**
1. API de email mudou
2. Formato de template mudou
3. Sistema de logs mudou 
4. Regras de validação mudaram

**Solução - cada classe uma responsabilidade:**

``` dart title:email_service.dart
class EmailService {
	Future<void> send(String to, String message) async {
		await ExternalEmailAPI.send(to, message);
	}
}
```

```dart title:message_formatter.dart
class MessageFormatter {
	String formatMessage(String template, Map<String, String> data) {
		return template.replaceAll('{{name}}', data['name'] ?? '');
	}
}
```

```dart title:notification_logger.dart
class NotificationLogger {
	void logNotification(String recipient, String status) {
		File('logs.txt').writeAsStringSync('$recipient - $status\n');
	}
}
```

```dart title:email_validator.dart
class EmailValidator {
	bool isValidEmail (String email) {
		return email.contains('@');
	}
}
```

```dart title:notification_orchestrator.dart
// Coordenador que usa os serviços
class NotificationOrchestrator {
	final EmailService emailService;
	final MessageFormatter formatter;
	final NotificationLogger logger;
	final EmailValidator validator;
	
	NotificationOrchestrator (
		this.emailService,
		this.formatter,
		this.logger,
		this.validator,
	);
	
	Future<void> sendNotification (
		String to,
		String template,
		Map<String, String> data,
	) async {
		if(!validator.isValid(to)) {
			logger.log(to, 'INVALID');
			throw Exception('Invalid email');
		}
		
		final message = formatter.format(template, data);
		await emailService.send(to, message);
		logger.log(to, 'SUCCESS');
	}
}
```

**Benefícios:**
- Se o sistema de logs mudar, só modifico `NotificationLogger`
- Posso testar validação independentemente de email
- Cada classe é pequena e focada/coesa.
- Mais fácil de reutilizar componentes

Para melhor consolidação veja no YouTube [  
SOLID: O Que Ninguém Te Explicou Sobre Responsabilidade Única!](https://www.youtube.com/watch?v=EWHTE1dQM4U&list=PLNHxHgB-_LTtxnX6ILHDBpY6hCuzBwlrW&index=5) vídeo explicativo sobre SRP.

[^1]: Um módulo pela definição mais simples, é apenas um arquivo fonte.
