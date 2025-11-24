No [[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)] o Uncle Bob menciona que o princípio Aberto/Fechado foi cunhado em 1988 por Bertrand Meyer[^1] e diz o seguinte:

> *Um artefato de software deve ser aberto para extensão, mas fechado para modificação*

Deves poder adicionar novo comportamento sem modificar código existente. Usa abstrações e polimorfismo para estender funcionalidade.

**Use Case - Sistema de notificações:**

``` dart title:notification_sender.dart
// VIOLACAO OCP - preciso modificar codigo existente para adicionar canais
class NotificationSender {
	void send(String type, String recipient, String message) {
		if (type == 'email') {
			//enviar email
		} else if (type == 'sms') {
			//enviar SMS
		} else if (type == 'whatsapp') { // <- Adicionei e modifiquei a classe
			//enviar WhatsApp 
		}
		// Para adicionar Telegram, preciso modificar AQUI novamenter 
	}
}
```


**Solução - extensível sem modificação:**
``` dart title:notification_channel_interface.dart
abstract class NotificationChannel {
	Future<void> send(String recipient, String message);
}
```

```dart title:email_channel.dart
class EmailChannel implements NotificationChannel {
	@override
	Future <void> send(String recipient, String message) async {
		// logica de email
	}
}
```

```dart title:sms_channel.dart
class SMSChannel implements NotificationChannel {
	@override
	Future<void> send(String recipient, String message) async {
		// logica de sms
	}
}
```

```dart title:whatsapp_channel.dart
// NOVA funcionalidade - SEM modificar codigo existente
class WhatsAppChannel implements NotificationChannel {
	@override
	Future<void> send(String recipient, String message) async {
		// logica de whatsapp
	}
}
```

[^1]: Bertrand Meyer. Object Oriented Software Construction, Prentice Hall, 1988, p.23.
