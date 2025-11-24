## #Encapsulamento 

segundo [claude.ai](https://claude.ai) é o processo de ocultar detalhes internos de implementação e expor apenas o necessário através de uma interface pública.

**Propósito:**
- proteger dados de acesso indevido
- reduzir acoplamento (dependências entre componentes)
- facilitar manutenção (mudanças internas não afetam código externo)

Exemplo - Sistema de notificações: 
``` dart fold="email_service".dart"
// sem encapsulamento
class EmailService {
	String apiKey = "123abc"; // exposto
	HttpClient client = HttpClient(); // Detalhes de implementacao expostos
	
	void sendEmail (String to, String message) {
		client.post(apiKey, to, message);
	}
}

// com encapsulamento
class EmailService {
	final String _apiKey; // Privado
	final HttpClient _client;
	
	EmailService(this._apyKey, this._client);
	
	// Interface publica simples
	Future <void> sendEmail(String to, String message) async {
		await _client.post(_apiKey, to, message);
	}
}
```

**Benefícios observados:** 
- Posso mudar o `HttpClient` internamente sem afetar quem usa `sendEmail()` 
- A API key está protegida
- Interface pública é simples e clara

---
## #Abstração 

segundo [claude.ai](https://claude.ai) é o processo de extrair características essenciais de um conceito, ignorando detalhes específicos de implementação.

**Propósito:**
- criar contratos/interfaces que definem "o quê" sem definir "como"
- permitir múltiplas implementações da mesma abstração
- facilitar testes e substituição de componentes

**Exemplo - Sistema de notificações:** 
``` dart fold="sistema_de_notificacoes"
// ABSTRACAO: define O QUE deve ser feito
abstract class NotificationService {
	Future<void> send(String recipient, String message);
}

// IMPLEMENTACOES: Definem COMO fazer
class EmailNotificationService implements NotificationService {
	@override
	Future<void> send (String recipient, String message) async {
		// Logica especifica de email
		await _sendViaSmtp(recipient, message);
	}
}

class SMSNotificationService implements NotificationService {
	@override
	Future<void> send(String recipient, String message) async {
		//Logica especifica de sms
		await _sendViaTwilio(recipient, message);
	}
}

class WhatsappNotificationService implements NotificationService {
	@override
	Future<void> send(String recipient, String message) async {
		// logica especifica de whastapp
		await _sendViaWhatsappAPI(recipient, message);
	}
}
```

**Benefícios observados:**
- posso adicionar novos canais (WhatsApp, Telegram) sem mudar código existente
- código cliente depende da abstração, não de implementações concretas
- facilita testes (posso criar `FakeNotificationService()`)
___
## #Herança 

segundo [claude.ai](https://claude.ai)é um mecanismo que permite criar classes derivadas (filhas) que herdam comportamentos e atributos de classes base (pais)

**Propósito:** 
- reutilizar código comum
- estabelecer relações "é-um" (is-a relationship)
- criar hierarquias de tipos

> [!ATENÇÃO]
> herança pode criar acoplamento forte.
>  
> Uncle Bob defende:
> *"Favoreça composição sobre herança"*
> 
> 

**Exemplo - Sistema de notificações:**
``` dart title:notification_service.dart 
// Classe base com comportamento comum
abstract class BaseNotificationService {
  final Logger logger;
  final MetricsCollector metrics;
  
  BaseNotificationService(this.logger, this.metrics);
  
  // Template Method Pattern
  Future<void> send(String recipient, String message) async {
    logger.info('Sending notification to $recipient');
    metrics.increment('notifications_sent');
    
    try {
      await sendImplementation(recipient, message);
      metrics.increment('notifications_success');
    } catch (e) {
      logger.error('Failed to send: $e');
      metrics.increment('notifications_failed');
      rethrow;
    }
  }
  
  // Metodo abstrato - cada filho implementa
  Future<void> sendImplementation(String recipient, String message);
}

// Classes derivadas
class EmailService extends BaseNotificationService {
  EmailService(super.logger, super.metrics);
  
  @override
  Future<void> sendImplementation(String recipient, String message) async {
    // Lógica especifica de email
  }
}
```

Problemas com herança excessiva:

- Hierarquias profundas são difíceis de entender
- Mudanças na classe base afetam todas as filhas
- Difícil prever efeitos colaterais
---
## #Polimorfismo

Segundo [claude.ai](https://claude.ai) é a capacidade de objetos de diferentes tipos responderem à mesma mensagem/método de formas diferentes.

**Propósito:**
- Escrever código genérico que funciona com múltiplos tipos
- Substituir implementações em runtime
- Inverter dependências (depender de abstrações)

**Exemplo - Sistema de notificações:**

``` dart title:notification_manager.dart
class NotificationManager {
  final NotificationService service;
  
  NotificationManager(this.service);
  
  // Este metodo funciona com QUALQUER NotificationService
  Future<void> notifyUser(String user, String message) async {
    await service.send(user, message);
    // Não sabe se eh Email, SMS ou WhatsApp - polimorfismo!
  }
}

// Em runtime, podemos decidir qual implementacao usar
void main() {
  NotificationManager emailManager = NotificationManager(EmailService());
  NotificationManager smsManager = NotificationManager(SMSService());
  
  // Mesmo método, comportamentos diferentes
  emailManager.notifyUser('user@email.com', 'Hello!');
  smsManager.notifyUser('+258823456789', 'Hello!');
}
```
--- 
2. [[Princípios SOLID explicados com use cases]]