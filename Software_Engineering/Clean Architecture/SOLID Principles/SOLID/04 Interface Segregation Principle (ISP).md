> *"Clientes não devem ser forçados a depender de interface que não usam"*

**Explicação:** é melhor ter várias interfaces específicas do que uma interface geral com muitos métodos. Classes não devem implementar métodos que não precisam.

**Use Case - Sistemas de notificações**

```dart title:notification_service.dart
// VIOLACAO ISP - interface "gorda"
abstract class NotificationService {
	Future<void> send(String recipient, String message);
	Future<void> sendBatch(List<String> recipients, String message);
	Future<void> schedule(String recipient, String message, DateTime time);
	Future<List<NotificationStatus>> getHistory();
	Future<void> cancelScheduled(String notificationId);
}
```

```dart title:email_service.dart
// EmailService precisa implementar TUDO, mesmo que nao suporte agendamento

class SimpleEmailService implements NotificationService {
	@override
	Future<void> send(String recipient, String Message) async {
		// Implementado
	}
	
	@override
	Future<void> sendBatch(List<String> recipients, String message) async {
		// Implementado
	}
	
	// Nao suporta agendamento, mas sou FORCADO a implementar
	@override
	Future<void> schedule (String recipient, String message, DateTime time) async {
		throw UnipmlementedError('Agendamento nao suportado');
	}
	
	// Tambem forcado a implementar
	@override
	Future<List<NotificationStatus>> getHistory() async {
		throw UniplementedError('Historico nao suportado');
	}
	
	@override
	Future<void> cancelScheduled(String notificationId) async {
		throw UnimplementedError('Cancelamento n\ao suportado');
	}
}
```

**SOLUÇÃO - Respeitar ISP(interfaces segregadas):**
```dart title:notification_sender_interface.dart
abstract class NotificationSender {
	Future<void> send(String recipient, String message);
}
```

```dart title:batch_notification_interface.dart
abstract class BatchNotificationSender {
	Future<void> sendBatch(List<String> recipients, String message);
}
```

```dart title: schedule_notification_interface.dart
abstract class SchedulableNotificationSender {
	Future<void> schedule(String recipient, String message, DateTime time);
	Future<void> cancelSchedule(String notificationId);
}
```

```dart title:notification_history_interface.dart
abstract class NotificationHistoryProvider {
	Future<List<NotificationStatus>> getHistory();
}
```

> Implementação simples - só implementação do que precisa:

```dart title:simple_email_service.dart
class SimpleEmailService implements NotificationSender {
	@override
	Future<void> send(String recipient, String message) async {
		// Implementacao
	}
}
```

> Implementação completa - implementação de todas as interfaces que suporta:

```dart title:advanced_email_service.dart
class AdvancedEmailService implements 
	NotificationSender,
	BatchNotificationSender,
	SchedulableNotificationSender,
	NotificationHistoryProvider {
		
		@override
		Future<void> send(String recipient, String message) async {
			// Implementacao
		}
		
		@override
		Future<void> sendBatch(List<String> recipients, String message) async {
			// Implementacao
		}
		
		@override
		Future<void> schedule(String recipient, String message, DateTime time) async {
			// Implementacao
		}
		
		@override
		Future<void> cancelSchedule(String NotificationId) async {
			// Implementacao
		}
		
		@override
		Future <List<NotificationStatus>> getHistory() async {
			// Implementacao
		}
	}
```

> Cliente depende apenas do que precisa:

```dart title:notification_controller.dart
class NotificationController {
	final NotificationSender sender; // so precisa enviar
	
	NotificationController(this. sender);
	
	Future<void>notify(String recipient, String message) async {
		await sender.send(recipient, message);
	}
}
```

**Benefícios:**
 - Classes implementam apenas o necessário
 - Interfaces focadas e coesas
 - Fácil de criar implementações
 - Clientes não dependem de métodos que não usam