
## Índice
- [Parte 1: Clean Architecture - Fundamentos](#fundamentos)
- [Parte 2: Clean Architecture em Flutter](#flutter-adaptation)
- [Parte 3: Da Estrutura MVC para Clean Architecture](#migracao-mvc)
- [Parte 4: Feature Completa - Sistema de Notificações](#feature-completa)

---
## <a id="fundamentos"></a> Parte 1: Clean Architecture - Fundamentos (Uncle Bob)

### O Problema que Clean Architecture resolve

**Uncle Bob pergunta:** "Como criar sistemas que sejam:"
- **Independentes de frameworks** - não casados com Flutter, React, Angular
- **Testáveis** - lógica de negócio testável sem UI, BD, servidor
- **Independentes de UI** - UI pode mudar sem afetar regras de negócio
- **Independentes de Database** - pode trocar PostgreSQL por MongoDB
- **Independentes de qualquer agente externo** - regras de negócio não sabem nada do mundo exterior

### A Solução:  Arquitetura em camadas concêntricas
![[clean-architecture-illustration.jpg]]
### As 4 Camadas explicadas

#### 1. Entities (Amarelo) - Centro do Sistema
**"Enterprise Business Rules"**

**O que são:**
- Regras de negócio mais gerais e de alto nível
- Objetos que encapsulam dados + comportamentos críticos do negócio
- Poderiam ser compartilhados entre múltiplas aplicações da empresa
- **NÃO mudam** quando a UI ou BD mudam

**Características:**
- Sem dependências externas (nem de framework, nem de camadas superiores)
- Puro Dart (ou qualquer linguagem)
- Mais estável de todas as camadas

**Exemplo - Sistema de Notificações:**
```dart
// Entity: Regra de negocio fundamental
class Notification {
  final String id;
  final String userId;
  final String title;
  final String message;
  final DateTime createdAt;
  final NotificationPriority priority;
  bool isRead;

  Notification({
    required this.id,
    required this.userId,
    required this.title,
    required this.message,
    required this.createdAt,
    required this.priority,
    this.isRead = false,
  });

  // Regra de negocio: notificacao expira apos 30 dias
  bool get isExpired {
    final now = DateTime.now();
    final difference = now.difference(createdAt);
    return difference.inDays > 30;
  }

  // Regra de negocio: marcar como lida
  void markAsRead() {
    if (isExpired) {
      throw NotificationExpiredException();
    }
    isRead = true;
  }

  // Regra de negocio: validacao
  bool get isValid {
    return title.isNotEmpty && 
           message.isNotEmpty && 
           userId.isNotEmpty;
  }
}

enum NotificationPriority {
  low,
  normal,
  high,
  urgent
}
```

**Uncle Bob diz:** "Entities não sabem nada sobre como são armazenadas, exibidas ou transmitidas. Elas só sabem as regras do negócio."

---
#### 2. Use Cases (Vermelho) - Casos de Uso da Aplicação
**"Application Business Rules"**

**O que são:**
- Orquestram o fluxo de dados de/para Entities
- Contêm regras específicas da aplicação
- Coordenam quando e como as Entities são usadas
- **Descrevem as intenções do usuário**

**Características:**
- Dependem apenas de Entities (camada interna)
- Não conhecem UI, BD, APIs (camadas externas)
- Um Use Case = Uma ação do usuário

**Exemplo - Sistema de Notificações:**
```dart
// Use Case: Representa uma acao especifica do usuario
class SendNotificationUseCase {
  final NotificationRepository repository; // ← Interface (abstracao)
  
  SendNotificationUseCase(this.repository);
  
  // Execute: metodo principal do Use Case
  Future<Result<Notification>> execute({
    required String userId,
    required String title,
    required String message,
    required NotificationPriority priority,
  }) async {
    try {
      // 1. Criar Entity (aplicar regras de negocio)
      final notification = Notification(
        id: _generateId(),
        userId: userId,
        title: title,
        message: message,
        createdAt: DateTime.now(),
        priority: priority,
      );
      
      // 2. Validar usando regras da Entity
      if (!notification.isValid) {
        return Result.failure(ValidationException('Dados invalidos'));
      }
      
      // 3. Persistir atraves do repositorio (abstracao)
      await repository.save(notification);
      
      // 4. Enviar atraves do canal apropriado
      await _sendThroughChannel(notification);
      
      return Result.success(notification);
    } catch (e) {
      return Result.failure(e);
    }
  }
  
  Future<void> _sendThroughChannel(Notification notification) async {
    // Logica de escolha de canal baseada em prioridade
    // Mas ainda nao sabe COMO enviar (isso eh camada externa)
  }
  
  String _generateId() => DateTime.now().millisecondsSinceEpoch.toString();
}

// Outro Use Case
class GetUserNotificationsUseCase {
  final NotificationRepository repository;
  
  GetUserNotificationsUseCase(this.repository);
  
  Future<Result<List<Notification>>> execute(String userId) async {
    try {
      final notifications = await repository.getByUserId(userId);
      
      // Regra de aplicacao: filtrar expiradas
      final validNotifications = notifications
          .where((n) => !n.isExpired)
          .toList();
      
      // Regra de aplicacao: ordenar por prioridade e data
      validNotifications.sort((a, b) {
        if (a.priority != b.priority) {
          return b.priority.index.compareTo(a.priority.index);
        }
        return b.createdAt.compareTo(a.createdAt);
      });
      
      return Result.success(validNotifications);
    } catch (e) {
      return Result.failure(e);
    }
  }
}
```

**Uncle Bob diz:** "Use Cases orquestram a dança dos dados entre as Entities e os limites externos."

**Características dos Use Cases:**
- Nome descritivo da ação: `SendNotification`, `GetUserNotifications`, `MarkNotificationAsRead`
- Um único método `execute()` ou `call()`
- Retornam `Result<T>` ou `Either<Failure, Success>` (error handling explícito)
- Não fazem I/O diretamente (delegam para repositórios)

---
#### 3. Interface Adapters (Verde) - Adaptadores
**"Conversores entre formatos"**

**O que são:**
- Convertem dados do formato mais conveniente para Use Cases/Entities para o formato mais conveniente para agentes externos
- Contêm Controllers, Presenters, Gateways
- **Implementam as interfaces** definidas pelos Use Cases

**Características:**
- Adaptam dados entre camadas
- Implementam abstrações (interfaces) definidas nas camadas internas
- Conhecem tanto Use Cases quanto Frameworks

**Componentes principais:**
##### A) Repositories (Implementação)
```dart
// INTERFACE definida na camada de Use Cases
abstract class NotificationRepository {
  Future<void> save(Notification notification);
  Future<List<Notification>> getByUserId(String userId);
  Future<Notification?> getById(String id);
  Future<void> delete(String id);
}

// IMPLEMENTACAO na camada de Interface Adapters
class NotificationRepositoryImpl implements NotificationRepository {
  final NotificationRemoteDataSource remoteDataSource;
  final NotificationLocalDataSource localDataSource;
  final NetworkInfo networkInfo;
  
  NotificationRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.networkInfo,
  });
  
  @override
  Future<void> save(Notification notification) async {
    try {
      // Converter Entity para Data Model (DTO)
      final model = NotificationModel.fromEntity(notification);
      
      // Tentar salvar remotamente
      if (await networkInfo.isConnected) {
        await remoteDataSource.createNotification(model);
      }
      
      // Sempre salvar localmente (cache)
      await localDataSource.cacheNotification(model);
    } catch (e) {
      throw RepositoryException('Falha ao salvar notificacao: $e');
    }
  }
  
  @override
  Future<List<Notification>> getByUserId(String userId) async {
    try {
      if (await networkInfo.isConnected) {
        // Buscar do servidor
        final models = await remoteDataSource.getNotificationsByUserId(userId);
        
        // Atualizar cache
        await localDataSource.cacheNotifications(models);
        
        // Converter Models para Entities
        return models.map((m) => m.toEntity()).toList();
      } else {
        // Sem internet: buscar do cache
        final models = await localDataSource.getCachedNotifications(userId);
        return models.map((m) => m.toEntity()).toList();
      }
    } catch (e) {
      throw RepositoryException('Falha ao buscar notificacoes: $e');
    }
  }
  
  @override
  Future<Notification?> getById(String id) async {
    // Implementacao similar
  }
  
  @override
  Future<void> delete(String id) async {
    // Implementacao similar
  }
}
```

##### B) Data Sources
```dart
// Data Source: lida com detalhes tecnicos de comunicacao
abstract class NotificationRemoteDataSource {
  Future<List<NotificationModel>> getNotificationsByUserId(String userId);
  Future<void> createNotification(NotificationModel notification);
}

class NotificationRemoteDataSourceImpl implements NotificationRemoteDataSource {
  final http.Client client;
  final String baseUrl;
  
  NotificationRemoteDataSourceImpl(this.client, this.baseUrl);
  
  @override
  Future<List<NotificationModel>> getNotificationsByUserId(String userId) async {
    final response = await client.get(
      Uri.parse('$baseUrl/notifications?userId=$userId'),
    );
    
    if (response.statusCode == 200) {
      final List<dynamic> jsonList = json.decode(response.body);
      return jsonList.map((json) => NotificationModel.fromJson(json)).toList();
    } else {
      throw ServerException();
    }
  }
  
  @override
  Future<void> createNotification(NotificationModel notification) async {
    final response = await client.post(
      Uri.parse('$baseUrl/notifications'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode(notification.toJson()),
    );
    
    if (response.statusCode != 201) {
      throw ServerException();
    }
  }
}
```

##### C) Models (DTOs - Data Transfer Objects)
```dart
// Model: representa dados como vem/vao para APIs/BD
class NotificationModel {
  final String id;
  final String userId;
  final String title;
  final String message;
  final String createdAt; // ← String, como vem da API
  final String priority;  // ← String, como vem da API
  final bool isRead;
  
  NotificationModel({
    required this.id,
    required this.userId,
    required this.title,
    required this.message,
    required this.createdAt,
    required this.priority,
    required this.isRead,
  });
  
  // Converter de JSON (API)
  factory NotificationModel.fromJson(Map<String, dynamic> json) {
    return NotificationModel(
      id: json['id'],
      userId: json['user_id'],
      title: json['title'],
      message: json['message'],
      createdAt: json['created_at'],
      priority: json['priority'],
      isRead: json['is_read'] ?? false,
    );
  }
  
  // Converter para JSON (API)
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'user_id': userId,
      'title': title,
      'message': message,
      'created_at': createdAt,
      'priority': priority,
      'is_read': isRead,
    };
  }
  
  // Converter Model → Entity (camada interna)
  Notification toEntity() {
    return Notification(
      id: id,
      userId: userId,
      title: title,
      message: message,
      createdAt: DateTime.parse(createdAt), // ← Conversao
      priority: _parsePriority(priority),     // ← Conversao
      isRead: isRead,
    );
  }
  
  // Converter Entity → Model (camada externa)
  factory NotificationModel.fromEntity(Notification notification) {
    return NotificationModel(
      id: notification.id,
      userId: notification.userId,
      title: notification.title,
      message: notification.message,
      createdAt: notification.createdAt.toIso8601String(), // ← Conversao
      priority: notification.priority.name,                 // ← Conversao
      isRead: notification.isRead,
    );
  }
  
  static NotificationPriority _parsePriority(String priority) {
    return NotificationPriority.values.firstWhere(
      (p) => p.name == priority,
      orElse: () => NotificationPriority.normal,
    );
  }
}
```

**Uncle Bob diz:** "Nenhum dado deve atravessar uma fronteira de forma inadequada. Conversões acontecem nos adaptadores."

---
#### 4. Frameworks & Drivers (Azul) - Camada Externa
**"Detalhes"**

**O que são:**
- Frameworks (Flutter, Firebase)
- UI (Widgets)
- Database (SQLite, Hive)
- APIs externas
- Devices (câmera, GPS)

**Características:**
- Camada mais instável (muda frequentemente)
- Contém código "sujo" (específico de framework)
- Depende de TODAS as outras camadas

**Exemplo - Sistema de Notificações:**
```dart
// UI (Widget) - Framework Layer
class NotificationsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Observa estado gerenciado pelo Provider/Riverpod
    final notificationsState = ref.watch(notificationsProvider);
    
    return Scaffold(
      appBar: AppBar(title: Text('Notificacoes')),
      body: notificationsState.when(
        loading: () => CircularProgressIndicator(),
        error: (error, _) => Text('Erro: $error'),
        data: (notifications) => ListView.builder(
          itemCount: notifications.length,
          itemBuilder: (context, index) {
            final notification = notifications[index];
            return NotificationTile(notification: notification);
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _sendTestNotification(ref),
        child: Icon(Icons.add),
      ),
    );
  }
  
  void _sendTestNotification(WidgetRef ref) {
    // Chama o controller que executa Use Case
    ref.read(notificationsProvider.notifier).sendNotification(
      title: 'Teste',
      message: 'Notificacao de teste',
      priority: NotificationPriority.normal,
    );
  }
}
```

---

### A Regra de Dependência (Dependency Rule)

**REGRA FUNDAMENTAL:** 
> *"Dependências de código-fonte devem apontar APENAS para DENTRO, em direção a políticas de mais alto nível."*

**O que isso significa:**
```
Camada Externa  →  Camada Interna
(PODE depender)

Camada Interna  ✗  Camada Externa
(NUNCA pode depender)
```

**Exemplos:**
- Use Case pode depender de Entity
- Repository pode depender de Use Case (interface)
- Widget pode depender de Use Case
- Entity NÃO pode depender de Use Case
- Use Case NÃO pode depender de Repository concreto
- Entity NÃO pode depender de Flutter

**Como garantir isso?**
Usando **ABSTRAÇÕES** (Dependency Inversion Principle - o "D" do SOLID!)

```dart
// ERRADO: Use Case depende de implementacao concreta (camada externa)
class SendNotificationUseCase {
  final NotificationRepositoryImpl repository; // ← Concreto!
}

// CORRETO: Use Case depende de abstracao (mesma camada ou interna)
class SendNotificationUseCase {
  final NotificationRepository repository; // ← Interface/Abstracao!
}
```

---
### Fluxo de Dados Completo

**Exemplo:** Usuário clica em "Enviar Notificação"

```
1. UI (Widget)
   ↓ chama
2. Controller/Presenter (Provider/Riverpod)
   ↓ executa
3. Use Case (SendNotificationUseCase)
   ↓ cria/valida
4. Entity (Notification)
   ↓ persiste através de
5. Repository (Interface - definida em Use Case)
   ↑ implementada por
6. Repository (Implementação - camada Adapters)
   ↓ usa
7. Data Source (Remote/Local)
   ↓ acessa
8. API/Database (Framework Layer)
```

**Fluxo de volta (resposta):**
```
8. API retorna dados
   ↓
7. Data Source recebe
   ↓
6. Repository converte Model → Entity
   ↓
5. Repository retorna Entity para Use Case
   ↓
4. Use Case retorna Result<Entity>
   ↓
3. Controller atualiza estado
   ↓
2. UI reage à mudança de estado
   ↓
1. Widget renderiza nova UI
```

---

### Benefícios da Clean Architecture

#### 1. Testabilidade
```dart
// Posso testar Use Case SEM banco de dados real
void main() {
  test('SendNotificationUseCase deve salvar notificacao valida', () async {
    // Arrange: Mock do repositorio
    final mockRepo = MockNotificationRepository();
    final useCase = SendNotificationUseCase(mockRepo);
    
    // Act: Executar Use Case
    final result = await useCase.execute(
      userId: '123',
      title: 'Teste',
      message: 'Mensagem teste',
      priority: NotificationPriority.normal,
    );
    
    // Assert: Verificar comportamento
    expect(result.isSuccess, true);
    verify(mockRepo.save(any)).called(1);
  });
}
```

#### 2. Independência de Framework
```dart
// Se amanhã trocar Flutter por React Native,
// só preciso reescrever a UI (camada externa)
// Use Cases, Entities permanecem IGUAIS!
```

#### 3. Facilidade de Manutenção
```dart
// Mudança na API? Só mexo no Data Source
// Nova regra de negócio? Só mexo na Entity/Use Case
// Nova UI? Só mexo nos Widgets
// Camadas isoladas = mudanças localizadas
```

#### 4. Substituição de Componentes
```dart
// Trocar SQLite por Hive? Só crio novo LocalDataSource
// Trocar API REST por GraphQL? Só crio novo RemoteDataSource
// Repository (interface) permanece igual!
```

## <a id="flutter-adaptation"></a> Parte 2: Clean Architecture em Flutter

### Estrutura de Pastas Feature-Based

```
lib/
├── core/
│   ├── errors/
│   │   ├── failures.dart
│   │   └── exceptions.dart
│   ├── usecases/
│   │   └── usecase.dart
│   ├── utils/
│   │   ├── network_info.dart
│   │   └── result.dart
│   └── constants/
│       └── app_constants.dart
│
└── features/
    └── notifications/
        ├── data/
        │   ├── datasources/
        │   │   ├── notification_remote_datasource.dart
        │   │   └── notification_local_datasource.dart
        │   ├── models/
        │   │   └── notification_model.dart
        │   └── repositories/
        │       └── notification_repository_impl.dart
        │
        ├── domain/
        │   ├── entities/
        │   │   └── notification.dart
        │   ├── repositories/
        │   │   └── notification_repository.dart
        │   └── usecases/
        │       ├── send_notification.dart
        │       ├── get_user_notifications.dart
        │       └── mark_notification_read.dart
        │
        └── presentation/
            ├── providers/
            │   └── notifications_provider.dart
            ├── screens/
            │   └── notifications_screen.dart
            └── widgets/
                ├── notification_tile.dart
                └── notification_filter.dart
```

### Mapeamento Camadas → Pastas

| Camada Clean Arch | Pasta Flutter | Contém |
|-------------------|---------------|---------|
| **Entities** | `domain/entities/` | Classes de negócio puras |
| **Use Cases** | `domain/usecases/` | Casos de uso da aplicação |
| **Repository Interface** | `domain/repositories/` | Contratos (interfaces) |
| **Repository Impl** | `data/repositories/` | Implementações dos contratos |
| **Data Sources** | `data/datasources/` | Acesso a APIs/BD |
| **Models** | `data/models/` | DTOs para serialização |
| **Controllers/Presenters** | `presentation/providers/` | Gerenciamento de estado |
| **UI** | `presentation/screens/` e `widgets/` | Interface visual |

### Convenções de Nomenclatura

```dart
// Entities: substantivos simples
class Notification { }
class User { }

// Use Cases: verbos + substantivo
class SendNotification { }
class GetUserNotifications { }
class MarkNotificationAsRead { }

// Repositories (interface): substantivo + Repository
abstract class NotificationRepository { }

// Repositories (impl): nome da interface + Impl
class NotificationRepositoryImpl implements NotificationRepository { }

// Data Sources: substantivo + Remote/Local + DataSource
abstract class NotificationRemoteDataSource { }
class NotificationRemoteDataSourceImpl { }

// Models: substantivo + Model
class NotificationModel { }

// Providers: substantivo + Provider/Notifier
class NotificationsProvider extends StateNotifier<NotificationsState> { }
```

---
## <a id="migracao-mvc"></a>Parte 3: De MVC para Clean Architecture

### Tua Estrutura Atual (MVC)

```
lib/
├── models/
│   └── notification.dart          // Entity + Model misturados
├── views/
│   └── notifications_screen.dart  // UI
├── controllers/
│   └── notification_controller.dart // Logica + API + Estado
├── services/
│   └── notification_service.dart   // Chamadas HTTP
└── core/
    └── utils.dart
```

### Problemas com MVC Tradicional

```dart
// Controller MVC tipico - TUDO misturado
class NotificationController extends ChangeNotifier {
  List<Notification> notifications = [];
  bool isLoading = false;
  
  // Problema 1: Controller conhece detalhes de HTTP
  Future<void> fetchNotifications(String userId) async {
    isLoading = true;
    notifyListeners();
    
    final response = await http.get(
      Uri.parse('https://api.com/notifications?userId=$userId'),
    );
    
    // Problema 2: Logica de conversão no controller
    if (response.statusCode == 200) {
      final List<dynamic> json = jsonDecode(response.body);
      notifications = json.map((j) => Notification.fromJson(j)).toList();
    }
    
    // Problema 3: Regras de negocio no controller
    notifications = notifications.where((n) => !n.isExpired).toList();
    
    isLoading = false;
    notifyListeners();
  }
  
  // Problema 4: Dificil testar sem fazer chamada HTTP real
}
```

**Consequências:**
- Controller faz TUDO (viola SRP)
- Impossível testar sem API real
- Mudança na API afeta controller
- Regras de negócio espalhadas
- Difícil reutilizar lógica

### Migração Step-by-Step
#### Step 1: Extrair Entity
```dart
// ANTES: model/notification.dart (Entity + Model misturados)
class Notification {
  String id;
  String userId;
  String title;
  Map<String, dynamic> toJson() { } // ← Conhece serializacao!
}

// DEPOIS: domain/entities/notification.dart (Entity pura)
class Notification {
  final String id;
  final String userId;
  final String title;
  final String message;
  final DateTime createdAt;
  final NotificationPriority priority;
  bool isRead;
  
  // Sem toJson/fromJson - Entity não conhece serializacao!
  
  bool get isExpired { } // ← Regras de negocio aqui
  void markAsRead() { }
}
```

#### Step 2: Criar Model (DTO)
```dart
// data/models/notification_model.dart
class NotificationModel {
  final String id;
  final String userId;
  // ... campos
  
  // Model CONHECE serializacao
  factory NotificationModel.fromJson(Map<String, dynamic> json) { }
  Map<String, dynamic> toJson() { }
  
  // Conversoes Model ↔ Entity
  Notification toEntity() { }
  factory NotificationModel.fromEntity(Notification entity) { }
}
```

#### Step 3: Extrair Data Source
```dart
// ANTES: service/notification_service.dart (HTTP direto)
class NotificationService {
  Future<List<Notification>> getNotifications(String userId) async {
    final response = await http.get(/*...*/);
    // ...
  }
}

// DEPOIS: data/datasources/notification_remote_datasource.dart
abstract class NotificationRemoteDataSource {
  Future<List<NotificationModel>> getNotificationsByUserId(String userId);
}

class NotificationRemoteDataSourceImpl implements NotificationRemoteDataSource {
  final http.Client client;
  
  @override
  Future<List<NotificationModel>> getNotificationsByUserId(String userId) async {
    final response = await client.get(/*...*/);
    
    if (response.statusCode == 200) {
      final List<dynamic> json = jsonDecode(response.body);
      return json.map((j) => NotificationModel.fromJson(j)).toList();
    } else {
      throw ServerException();
    }
  }
}
```

#### Step 4: Criar Repository
```dart
// domain/repositories/notification_repository.dart (Interface)
abstract class NotificationRepository {
  Future<List<Notification>> getByUserId(String userId);
  Future<void> save(Notification notification);
}

// data/repositories/notification_repository_impl.dart (Implementacao)
class NotificationRepositoryImpl implements NotificationRepository {
  final NotificationRemoteDataSource remoteDataSource;
  
  @override
  Future<List<Notification>> getByUserId(String userId) async {
    final models = await remoteDataSource.getNotificationsByUserId(userId);
    return models.map((m) => m.toEntity()).toList();
  }
}
```

#### Step 5: Criar Use Cases
```dart
// ANTES: Logica no controller
class NotificationController {
  Future<void> fetchNotifications(String userId) async {
    // Buscar + filtrar + ordenar TUDO aqui
  }
}

// DEPOIS: domain/usecases/get_user_notifications.dart
class GetUserNotifications {
  final NotificationRepository repository;
  
  GetUserNotifications(this.repository);
  
  Future<Result<List<Notification>>> call(String userId) async {
    try {
      final notifications = await repository.getByUserId(userId);
      
      // Regras de aplicacao aqui
      final validNotifications = notifications
          .where((n) => !n.isExpired)
          .toList();
      
      validNotifications.sort((a, b) => 
        b.priority.index.compareTo(a.priority.index)
      );
      
      return Result.success(validNotifications);
    } catch (e) {
      return Result.failure(Failure(e.toString()));
    }
  }
}
```

#### Step 6: Refatorar Controller → Provider (Riverpod)
```dart
// ANTES: controller/notification_controller.dart
class NotificationController extends ChangeNotifier {
  List<Notification> notifications = [];
  bool isLoading = false;
  
  Future<void> fetchNotifications(String userId) async {
    // TUDO aqui: HTTP + logica + estado
  }
}

// DEPOIS: presentation/providers/notifications_provider.dart
@riverpod
class NotificationsNotifier extends _$NotificationsNotifier {
  @override
  FutureOr<List<Notification>> build(String userId) async {
    // Executar Use Case
    final result = await ref.read(getUserNotificationsUseCaseProvider).call(userId);
    
    return result.when(
      success: (notifications) => notifications,
      failure: (error) => throw error,
    );
  }
  
  Future<void> sendNotification({
    required String title,
    required String message,
    required NotificationPriority priority,
  }) async {
    // Executar Use Case
    final result = await ref.read(sendNotificationUseCaseProvider).call(
      userId: state.value?.first.userId ?? '',
      title: title,
      message: message,
      priority: priority,
    );
    
    result.when(
      success: (_) => ref.invalidateSelf(), // Recarregar lista
      failure: (error) => throw error,
    );
  }
  
  Future<void> markAsRead(String notificationId) async {
    final result = await ref.read(markNotificationReadUseCaseProvider).call(notificationId);
    
    result.when(
      success: (_) => ref.invalidateSelf(),
      failure: (error) => throw error,
    );
  }
}

// Providers para injecao de dependencias
@riverpod
GetUserNotifications getUserNotificationsUseCase(GetUserNotificationsUseCaseRef ref) {
  return GetUserNotifications(ref.read(notificationRepositoryProvider));
}

@riverpod
SendNotification sendNotificationUseCase(SendNotificationUseCaseRef ref) {
  return SendNotification(ref.read(notificationRepositoryProvider));
}

@riverpod
NotificationRepository notificationRepository(NotificationRepositoryRef ref) {
  return NotificationRepositoryImpl(
    remoteDataSource: ref.read(notificationRemoteDataSourceProvider),
    localDataSource: ref.read(notificationLocalDataSourceProvider),
    networkInfo: ref.read(networkInfoProvider),
  );
}
```

### Comparação Lado a Lado

| Aspecto | MVC Tradicional | Clean Architecture |
|---------|----------------|-------------------|
| **Responsabilidades** | Controller faz tudo | Cada camada uma função |
| **Testabilidade** | Difícil (acoplado) | Fácil (injeção de dep.) |
| **Independência** | Acoplado ao Flutter | Lógica independente |
| **Manutenção** | Mudanças afetam tudo | Mudanças localizadas |
| **Reutilização** | Difícil | Use Cases reutilizáveis |
| **SOLID** | Viola vários | Respeita todos |

---
## <a id="feature-completa"></a>Parte 4: Feature Completa - Sistema de Notificações

### Estrutura Completa da Feature

```
features/notifications/
├── data/
│   ├── datasources/
│   │   ├── notification_remote_datasource.dart
│   │   └── notification_local_datasource.dart
│   ├── models/
│   │   └── notification_model.dart
│   └── repositories/
│       └── notification_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── notification.dart
│   ├── repositories/
│   │   └── notification_repository.dart
│   └── usecases/
│       ├── send_notification.dart
│       ├── get_user_notifications.dart
│       └── mark_notification_read.dart
└── presentation/
    ├── providers/
    │   └── notifications_provider.dart
    ├── screens/
    │   └── notifications_screen.dart
    └── widgets/
        ├── notification_tile.dart
        └── empty_notifications.dart
```

### Código Completo

#### 1. Domain Layer (Núcleo)

**entities/notification.dart**
```dart
class Notification {
  final String id;
  final String userId;
  final String title;
  final String message;
  final DateTime createdAt;
  final NotificationPriority priority;
  bool isRead;

  Notification({
    required this.id,
    required this.userId,
    required this.title,
    required this.message,
    required this.createdAt,
    required this.priority,
    this.isRead = false,
  });

  bool get isExpired {
    final now = DateTime.now();
    return now.difference(createdAt).inDays > 30;
  }

  void markAsRead() {
    if (isExpired) throw NotificationExpiredException();
    isRead = true;
  }

  bool get isValid {
    return title.isNotEmpty && message.isNotEmpty && userId.isNotEmpty;
  }
}

enum NotificationPriority { low, normal, high, urgent }
```

**repositories/notification_repository.dart** (Interface)
```dart
abstract class NotificationRepository {
  Future<List<Notification>> getByUserId(String userId);
  Future<void> save(Notification notification);
  Future<Notification?> getById(String id);
  Future<void> markAsRead(String id);
}
```

**usecases/get_user_notifications.dart**
```dart
class GetUserNotifications {
  final NotificationRepository repository;

  GetUserNotifications(this.repository);

  Future<Result<List<Notification>>> call(String userId) async {
    try {
      final notifications = await repository.getByUserId(userId);
      
      final validNotifications = notifications
          .where((n) => !n.isExpired)
          .toList();
      
      validNotifications.sort((a, b) {
        if (a.priority != b.priority) {
          return b.priority.index.compareTo(a.priority.index);
        }
        return b.createdAt.compareTo(a.createdAt);
      });
      
      return Result.success(validNotifications);
    } catch (e) {
      return Result.failure(Failure('Falha ao buscar notificacoes: $e'));
    }
  }
}
```

**usecases/send_notification.dart**
```dart
class SendNotification {
  final NotificationRepository repository;

  SendNotification(this.repository);

  Future<Result<Notification>> call({
    required String userId,
    required String title,
    required String message,
    required NotificationPriority priority,
  }) async {
    try {
      final notification = Notification(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        userId: userId,
        title: title,
        message: message,
        createdAt: DateTime.now(),
        priority: priority,
      );

      if (!notification.isValid) {
        return Result.failure(ValidationFailure('Dados invalidos'));
      }

      await repository.save(notification);
      
      return Result.success(notification);
    } catch (e) {
      return Result.failure(Failure('Falha ao enviar: $e'));
    }
  }
}
```

**usecases/mark_notification_read.dart**
```dart
class MarkNotificationAsRead {
  final NotificationRepository repository;

  MarkNotificationAsRead(this.repository);

  Future<Result<void>> call(String notificationId) async {
    try {
      await repository.markAsRead(notificationId);
      return Result.success(null);
    } catch (e) {
      return Result.failure(Failure('Falha ao marcar como lida: $e'));
    }
  }
}
```

#### 2. Data Layer (Implementação)

**models/notification_model.dart**
```dart
class NotificationModel {
  final String id;
  final String userId;
  final String title;
  final String message;
  final String createdAt;
  final String priority;
  final bool isRead;

  NotificationModel({
    required this.id,
    required this.userId,
    required this.title,
    required this.message,
    required this.createdAt,
    required this.priority,
    required this.isRead,
  });

  factory NotificationModel.fromJson(Map<String, dynamic> json) {
    return NotificationModel(
      id: json['id'],
      userId: json['user_id'],
      title: json['title'],
      message: json['message'],
      createdAt: json['created_at'],
      priority: json['priority'],
      isRead: json['is_read'] ?? false,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'user_id': userId,
      'title': title,
      'message': message,
      'created_at': createdAt,
      'priority': priority,
      'is_read': isRead,
    };
  }

  Notification toEntity() {
    return Notification(
      id: id,
      userId: userId,
      title: title,
      message: message,
      createdAt: DateTime.parse(createdAt),
      priority: NotificationPriority.values.firstWhere(
        (p) => p.name == priority,
        orElse: () => NotificationPriority.normal,
      ),
      isRead: isRead,
    );
  }

  factory NotificationModel.fromEntity(Notification notification) {
    return NotificationModel(
      id: notification.id,
      userId: notification.userId,
      title: notification.title,
      message: notification.message,
      createdAt: notification.createdAt.toIso8601String(),
      priority: notification.priority.name,
      isRead: notification.isRead,
    );
  }
}
```

**datasources/notification_remote_datasource.dart**
```dart
abstract class NotificationRemoteDataSource {
  Future<List<NotificationModel>> getNotificationsByUserId(String userId);
  Future<void> createNotification(NotificationModel notification);
  Future<void> markAsRead(String id);
}

class NotificationRemoteDataSourceImpl implements NotificationRemoteDataSource {
  final http.Client client;
  final String baseUrl;

  NotificationRemoteDataSourceImpl({
    required this.client,
    required this.baseUrl,
  });

  @override
  Future<List<NotificationModel>> getNotificationsByUserId(String userId) async {
    final response = await client.get(
      Uri.parse('$baseUrl/notifications?userId=$userId'),
      headers: {'Content-Type': 'application/json'},
    );

    if (response.statusCode == 200) {
      final List<dynamic> jsonList = json.decode(response.body);
      return jsonList.map((json) => NotificationModel.fromJson(json)).toList();
    } else {
      throw ServerException();
    }
  }

  @override
  Future<void> createNotification(NotificationModel notification) async {
    final response = await client.post(
      Uri.parse('$baseUrl/notifications'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode(notification.toJson()),
    );

    if (response.statusCode != 201) {
      throw ServerException();
    }
  }

  @override
  Future<void> markAsRead(String id) async {
    final response = await client.patch(
      Uri.parse('$baseUrl/notifications/$id'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode({'is_read': true}),
    );

    if (response.statusCode != 200) {
      throw ServerException();
    }
  }
}
```

**datasources/notification_local_datasource.dart**
```dart
abstract class NotificationLocalDataSource {
  Future<List<NotificationModel>> getCachedNotifications(String userId);
  Future<void> cacheNotifications(List<NotificationModel> notifications);
  Future<void> cacheNotification(NotificationModel notification);
}

class NotificationLocalDataSourceImpl implements NotificationLocalDataSource {
  final SharedPreferences sharedPreferences;
  static const CACHED_NOTIFICATIONS = 'CACHED_NOTIFICATIONS';

  NotificationLocalDataSourceImpl({required this.sharedPreferences});

  @override
  Future<List<NotificationModel>> getCachedNotifications(String userId) async {
    final jsonString = sharedPreferences.getString('${CACHED_NOTIFICATIONS}_$userId');
    
    if (jsonString != null) {
      final List<dynamic> jsonList = json.decode(jsonString);
      return jsonList.map((json) => NotificationModel.fromJson(json)).toList();
    } else {
      throw CacheException();
    }
  }

  @override
  Future<void> cacheNotifications(List<NotificationModel> notifications) async {
    if (notifications.isEmpty) return;
    
    final userId = notifications.first.userId;
    final jsonString = json.encode(
      notifications.map((n) => n.toJson()).toList(),
    );
    
    await sharedPreferences.setString('${CACHED_NOTIFICATIONS}_$userId', jsonString);
  }

  @override
  Future<void> cacheNotification(NotificationModel notification) async {
    try {
      final cached = await getCachedNotifications(notification.userId);
      cached.add(notification);
      await cacheNotifications(cached);
    } catch (e) {
      await cacheNotifications([notification]);
    }
  }
}
```

**repositories/notification_repository_impl.dart**
```dart
class NotificationRepositoryImpl implements NotificationRepository {
  final NotificationRemoteDataSource remoteDataSource;
  final NotificationLocalDataSource localDataSource;
  final NetworkInfo networkInfo;

  NotificationRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.networkInfo,
  });

  @override
  Future<List<Notification>> getByUserId(String userId) async {
    if (await networkInfo.isConnected) {
      try {
        final models = await remoteDataSource.getNotificationsByUserId(userId);
        await localDataSource.cacheNotifications(models);
        return models.map((m) => m.toEntity()).toList();
      } catch (e) {
        throw ServerException();
      }
    } else {
      try {
        final models = await localDataSource.getCachedNotifications(userId);
        return models.map((m) => m.toEntity()).toList();
      } catch (e) {
        throw CacheException();
      }
    }
  }

  @override
  Future<void> save(Notification notification) async {
    final model = NotificationModel.fromEntity(notification);
    
    try {
      if (await networkInfo.isConnected) {
        await remoteDataSource.createNotification(model);
      }
      await localDataSource.cacheNotification(model);
    } catch (e) {
      throw RepositoryException();
    }
  }

  @override
  Future<Notification?> getById(String id) async {
    // Implementacao similar
    throw UnimplementedError();
  }

  @override
  Future<void> markAsRead(String id) async {
    if (await networkInfo.isConnected) {
      await remoteDataSource.markAsRead(id);
    }
  }
}
```

#### 3. Presentation Layer (UI + Estado)

**providers/notifications_provider.dart** (Riverpod)
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'notifications_provider.g.dart';

@riverpod
class NotificationsNotifier extends _$NotificationsNotifier {
  @override
  FutureOr<List<Notification>> build(String userId) async {
    final result = await ref
        .read(getUserNotificationsUseCaseProvider)
        .call(userId);

    return result.when(
      success: (notifications) => notifications,
      failure: (error) => throw error.message,
    );
  }

  Future<void> sendNotification({
    required String title,
    required String message,
    required NotificationPriority priority,
  }) async {
    final userId = state.value?.first.userId ?? '';
    
    final result = await ref.read(sendNotificationUseCaseProvider).call(
          userId: userId,
          title: title,
          message: message,
          priority: priority,
        );

    result.when(
      success: (_) => ref.invalidateSelf(),
      failure: (error) => throw error.message,
    );
  }

  Future<void> markAsRead(String notificationId) async {
    final result = await ref
        .read(markNotificationReadUseCaseProvider)
        .call(notificationId);

    result.when(
      success: (_) => ref.invalidateSelf(),
      failure: (error) => throw error.message,
    );
  }
}

// Dependency Injection - Use Cases
@riverpod
GetUserNotifications getUserNotificationsUseCase(
  GetUserNotificationsUseCaseRef ref,
) {
  return GetUserNotifications(ref.read(notificationRepositoryProvider));
}

@riverpod
SendNotification sendNotificationUseCase(SendNotificationUseCaseRef ref) {
  return SendNotification(ref.read(notificationRepositoryProvider));
}

@riverpod
MarkNotificationAsRead markNotificationReadUseCase(
  MarkNotificationReadUseCaseRef ref,
) {
  return MarkNotificationAsRead(ref.read(notificationRepositoryProvider));
}

// Dependency Injection - Repository
@riverpod
NotificationRepository notificationRepository(
  NotificationRepositoryRef ref,
) {
  return NotificationRepositoryImpl(
    remoteDataSource: ref.read(notificationRemoteDataSourceProvider),
    localDataSource: ref.read(notificationLocalDataSourceProvider),
    networkInfo: ref.read(networkInfoProvider),
  );
}

// Dependency Injection - Data Sources
@riverpod
NotificationRemoteDataSource notificationRemoteDataSource(
  NotificationRemoteDataSourceRef ref,
) {
  return NotificationRemoteDataSourceImpl(
    client: ref.read(httpClientProvider),
    baseUrl: 'https://api.example.com',
  );
}

@riverpod
NotificationLocalDataSource notificationLocalDataSource(
  NotificationLocalDataSourceRef ref,
) {
  return NotificationLocalDataSourceImpl(
    sharedPreferences: ref.read(sharedPreferencesProvider),
  );
}
```

**screens/notifications_screen.dart**
```dart
class NotificationsScreen extends ConsumerWidget {
  final String userId;

  const NotificationsScreen({required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final notificationsAsync = ref.watch(notificationsNotifierProvider(userId));

    return Scaffold(
      appBar: AppBar(
        title: Text('Notificacoes'),
        actions: [
          IconButton(
            icon: Icon(Icons.filter_list),
            onPressed: () => _showFilterDialog(context),
          ),
        ],
      ),
      body: notificationsAsync.when(
        loading: () => Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.error_outline, size: 48, color: Colors.red),
              SizedBox(height: 16),
              Text('Erro: $error'),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: () => ref.invalidate(notificationsNotifierProvider),
                child: Text('Tentar Novamente'),
              ),
            ],
          ),
        ),
        data: (notifications) {
          if (notifications.isEmpty) {
            return EmptyNotifications();
          }

          return RefreshIndicator(
            onRefresh: () async {
              ref.invalidate(notificationsNotifierProvider(userId));
            },
            child: ListView.builder(
              itemCount: notifications.length,
              itemBuilder: (context, index) {
                final notification = notifications[index];
                return NotificationTile(
                  notification: notification,
                  onTap: () => _handleNotificationTap(ref, notification),
                );
              },
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showSendDialog(context, ref),
        child: Icon(Icons.add),
      ),
    );
  }

  void _handleNotificationTap(WidgetRef ref, Notification notification) {
    if (!notification.isRead) {
      ref
          .read(notificationsNotifierProvider(userId).notifier)
          .markAsRead(notification.id);
    }
  }

  void _showSendDialog(BuildContext context, WidgetRef ref) {
    // Dialog para enviar notificacao
  }

  void _showFilterDialog(BuildContext context) {
    // Dialog para filtrar notificacoes
  }
}
```

**widgets/notification_tile.dart**
```dart
class NotificationTile extends StatelessWidget {
  final Notification notification;
  final VoidCallback onTap;

  const NotificationTile({
    required this.notification,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      elevation: notification.isRead ? 0 : 2,
      child: ListTile(
        leading: _buildPriorityIcon(),
        title: Text(
          notification.title,
          style: TextStyle(
            fontWeight: notification.isRead ? FontWeight.normal : FontWeight.bold,
          ),
        ),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(notification.message),
            SizedBox(height: 4),
            Text(
              _formatDate(notification.createdAt),
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
          ],
        ),
        trailing: notification.isRead
            ? Icon(Icons.check_circle, color: Colors.green)
            : Icon(Icons.circle_outlined, color: Colors.grey),
        onTap: onTap,
      ),
    );
  }

  Widget _buildPriorityIcon() {
    final color = _getPriorityColor();
    final icon = _getPriorityIcon();
    
    return CircleAvatar(
      backgroundColor: color.withOpacity(0.2),
      child: Icon(icon, color: color),
    );
  }

  Color _getPriorityColor() {
    switch (notification.priority) {
      case NotificationPriority.urgent:
        return Colors.red;
      case NotificationPriority.high:
        return Colors.orange;
      case NotificationPriority.normal:
        return Colors.blue;
      case NotificationPriority.low:
        return Colors.grey;
    }
  }

  IconData _getPriorityIcon() {
    switch (notification.priority) {
      case NotificationPriority.urgent:
        return Icons.priority_high;
      case NotificationPriority.high:
        return Icons.arrow_upward;
      case NotificationPriority.normal:
        return Icons.notifications;
      case NotificationPriority.low:
        return Icons.arrow_downward;
    }
  }

  String _formatDate(DateTime date) {
    final now = DateTime.now();
    final difference = now.difference(date);

    if (difference.inDays == 0) {
      if (difference.inHours == 0) {
        return '${difference.inMinutes}m atras';
      }
      return '${difference.inHours}h atras';
    } else if (difference.inDays < 7) {
      return '${difference.inDays}d atras';
    } else {
      return '${date.day}/${date.month}/${date.year}';
    }
  }
}
```

---

## Resumo Final

### Checklist Clean Architecture

- **Entities** são puras (sem dependências externas)  
- **Use Cases** orquestram fluxo (uma ação = um use case)  
- **Repositories** são abstrações (interfaces na domain layer)  
- **Data Sources** lidam com detalhes técnicos  
- **Models** convertem entre camadas  
- **Presentation** só chama Use Cases (via Riverpod)  
- **Dependências** apontam para dentro  
- **Testável** (cada camada isolada)  

### Benefícios Conquistados

1. **Testabilidade** - Posso testar Use Cases sem UI/BD
2. **Manutenibilidade** - Mudanças localizadas por camada
3. **Escalabilidade** - Fácil adicionar novas features
4. **Independência** - Lógica não depende de Flutter
5. **SOLID** - Todos os princípios aplicados
6. **Clareza** - Estrutura clara e previsível

---

## Próximos Passos

1. **Praticar** - Implementar outra feature (ex: autenticação)
2. **Testar** - Escrever testes unitários para Use Cases
3. **Aprofundar Riverpod** - Explorar `AsyncNotifier`, `Family`, `AutoDispose`
4. **Error Handling** - Implementar `Either<Failure, Success>` ou `Result<T>`
5. **Code Generation** - Usar `freezed` para Entities/Models imutáveis

### Recursos

- **Livro**: Clean Architecture (Capítulos 19-23 sobre camadas)
- **Blog**: [blog.cleancoder.com](blog.cleancoder.com)
- **Vídeo**: ["Clean Architecture"](https://youtu.be/ow8UUjS5vzU?si=Fo2SJq6IvlD5tGwF) no YouTube