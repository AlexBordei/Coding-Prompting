
# 🧩 Flutter Clean Architecture with BLoC – Full Scaffold Prompt

Use this prompt to generate a **production-ready Flutter app** using **clean architecture**, **BLoC pattern**, **dependency injection**, and modern Flutter best practices.
Start by using flutter create command
---

## ✅ Requirements

Create a Flutter app scaffold that follows these specifications:

### 📁 Directory Structure

```
lib/
├── main.dart
├── core/
│   ├── network/                # Network-related helpers and clients (e.g., Dio client)
│   ├── error/                  # App-wide error definitions and handling
│   ├── usecases/              # Base usecase patterns
│   ├── constants/             # App constants and enums
│   ├── service/               # Reusable services
│   ├── style/                 # Theming, colors, typography
│   ├── widgets/               # Common widgets shared across app
│   └── service_locator.dart   # GetIt DI container
├── features/
│   └── {feature_name}/
│       ├── data/
│       │   ├── datasources/
│       │   ├── models/
│       │   └── repositories/
│       ├── domain/
│       │   ├── entities/
│       │   ├── repositories/
│       │   └── usecases/
│       └── presentation/
│           ├── bloc/
│           ├── pages/
│           └── widgets/
```

---

## 📦 Packages to Include

Add these dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  font_awesome_flutter: ^10.8.0
  intl: ^0.20.2
  bloc: ^9.0.0
  flutter_bloc: ^9.1.0
  dio: ^5.8.0+1
  get_it: ^8.0.3
  equatable: ^2.0.7
  meta: ^1.16.0
  flutter_dotenv: ^5.2.1
  internet_connection_checker_plus: ^2.7.1
```

---

## 🔄 Clean Architecture Layers

### 📘 Domain Layer
- Create **abstract repository interfaces**
- Create **pure Dart entity models**
- Add use cases that follow these patterns:

```dart
abstract class UseCase<Type, Params> {
  Future<Type> call(Params params);
}

abstract class VoidUseCase<Params> {
  Future<void> call(Params params);
}

abstract class NoParamsUseCase<Type> {
  Future<Type> call();
}
```

#### Example: `LoginUseCase`

```dart
class LoginUseCase extends UseCase<UserEntity, LoginParams> {
  final AuthRepository repository;

  LoginUseCase(this.repository);

  @override
  Future<UserEntity> call(LoginParams params) {
    return repository.login(params.email, params.password);
  }
}
```

---

### 💾 Data Layer
- Implement repository interfaces
- Include:
  - `RemoteDataSource`
  - `LocalDataSource` (optional)
  - `Model ↔ Entity` conversions
  - Error & network handling

```dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  final NetworkInfo networkInfo;

  AuthRepositoryImpl({
    required this.remoteDataSource,
    required this.networkInfo,
  });

  @override
  Future<UserEntity> login(String email, String password) async {
    if (await networkInfo.isConnected) {
      try {
        final user = await remoteDataSource.login(email, password);
        return user;
      } catch (e) {
        throw ServerException(message: e.toString());
      }
    } else {
      throw ServerException(message: "No internet connection");
    }
  }
}
```

---

### 🖼️ Presentation Layer
- Implement the **BLoC pattern** per feature.
- Use `flutter_bloc` with separated files for:
  - Bloc
  - Events
  - States

```dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginButtonPressed>((event, emit) async {
      emit(AuthLoading());
      try {
        final user = await sl<LoginUseCase>().call(LoginParams(
          email: event.email,
          password: event.password,
        ));
        emit(AuthSuccess(user));
      } catch (e) {
        emit(AuthFailure(e.toString()));
      }
    });
  }
}
```

---

## 🧪 Dependency Injection

Use `GetIt` for managing dependencies.

```dart
final sl = GetIt.instance;

Future<void> init() async {
  // Network Info
  sl.registerLazySingleton<NetworkInfo>(() => NetworkInfoImpl());

  // Data sources
  sl.registerLazySingleton<AuthRemoteDataSource>(() => AuthRemoteSourceImpl());

  // Repositories
  sl.registerLazySingleton<AuthRepository>(() => AuthRepositoryImpl(
        remoteDataSource: sl(),
        networkInfo: sl(),
      ));

  // Use cases
  sl.registerLazySingleton(() => LoginUseCase(sl()));
}
```

---

## 🌐 Network Layer
- Use `dio` with base options and interceptors
- Implement error handling, headers, timeout logic
- Centralize `DioClient` or `ApiService`

---

## 🚨 Error Handling
- Create `Failure` and `Exception` classes
- Consistently propagate exceptions from data → domain → presentation

```dart
class ServerException implements Exception {
  final String message;
  ServerException({required this.message});
}
```

---

## 🎨 Theme and Design System

Define centralized theming:

```
core/style/
├── app_colors.dart
├── app_text_styles.dart
├── app_theme.dart
```

- Support **light/dark mode**
- Use `ThemeData` for easy customization

---

## 🌐 Connectivity Checks

Use `internet_connection_checker_plus`:

```dart
abstract class NetworkInfo {
  Future<bool> get isConnected;
}

class NetworkInfoImpl implements NetworkInfo {
  final InternetConnectionCheckerPlus connectionChecker;

  NetworkInfoImpl(this.connectionChecker);

  @override
  Future<bool> get isConnected => connectionChecker.hasConnection;
}
```

---

## 🌱 Feature Example: Auth

Scaffold a `features/auth/` folder with:

- `data/`: RemoteDataSource, AuthModel
- `domain/`: UserEntity, AuthRepository, LoginUseCase
- `presentation/`: AuthBloc, AuthEvent, AuthState, LoginPage

---

## 🧪 Testing Ready

- Use `equatable` for value equality
- Use `mockito` or `mocktail` for testing
- DI and architecture enable test-driven development

---

## 🧾 .env Support

Add `.env` loading in `main.dart`:

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: ".env");
  await init(); // DI
  runApp(MyApp());
}
```
