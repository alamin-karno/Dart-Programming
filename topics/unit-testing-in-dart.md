# Unit Testing in Dart

Unit testing is a fundamental practice in software development, ensuring that individual components of your code function as intended. In Dart, the `test` package provides a robust framework for writing and running unit tests. This guide will walk you through setting up unit tests in Dart, writing tests for both synchronous and asynchronous code, and employing best practices to enhance your testing strategy.

### Setting Up Your Testing Environment

1.  **Add the `test` package to your project:**

    In your `pubspec.yaml` file, include the `test` package under `dev_dependencies`:

```yaml
dev_dependencies:
  test: ^1.21.0
```

Then, run `dart pub get` to install the package.

2.  **Create a `test` directory:**

    At the root of your project, create a `test` folder to store your test files. Ensure that your test files end with `_test.dart` to be recognized by the test runner.

### Understanding the AAA Pattern

The **AAA (Arrange, Act, Assert)** pattern is a widely adopted structure for writing clear and concise tests:

* **Arrange:** Set up the necessary objects and prepare the prerequisites for your test.
* **Act:** Execute the function or method under test.
* **Assert:** Verify that the outcome matches your expectations.

This pattern promotes readability and maintainability in your test code.

***

### Writing a Basic Unit Test

Let's start with a simple example of testing a function that adds two numbers:

**`lib/calculator.dart`**

```dart
int add(int a, int b) {
  return a + b;
}
```

`test/calculator_test.dart`

```dart
import 'package:test/test.dart';
import '../lib/calculator.dart';

void main() {
  test('adds two numbers', () {
    // Arrange
    int a = 2;
    int b = 3;

    // Act
    int result = add(a, b);

    // Assert
    expect(result, equals(5));
  });
}
```

In this test, we verify that the `add` function correctly returns the sum of two numbers.

Output:

{% hint style="success" %}
00:00 +1: All tests passed!
{% endhint %}

This test passes because `add(2, 3)` correctly returns `5`.

***

### Testing Asynchronous Code

Dart's `Future` and `Stream` classes are commonly used for asynchronous operations. Testing such code requires the use of `async` and `await` in your test functions.

**Example: Testing a function that fetches data asynchronously**

**`lib/data_fetcher.dart`**

```dart
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Data loaded';
}
```

`test/data_fetcher_test.dart`

```dart
import 'package:test/test.dart';
import '../lib/data_fetcher.dart';

void main() {
  test('fetchData returns expected string', () async {
    // Act
    String result = await fetchData();

    // Assert
    expect(result, equals('Data loaded'));
  });
}
```

This test ensures that the `fetchData` function returns the expected string after completing its asynchronous operation.

Output (after \~1 second delay):

{% hint style="success" %}
00:01 +1: All tests passed!
{% endhint %}

This test passes after the `Future.delayed` completes and returns `'Data loaded'`.

***

### Using Matchers for More Expressive Tests

Dart's `test` package provides a rich set of matchers to write expressive and readable assertions.

**Example:**

```dart
test('value is not null and greater than zero', () {
  int value = 10;

  expect(value, isNotNull);
  expect(value, greaterThan(0));
});
```

Using matchers like `isNotNull` and `greaterThan` enhances the clarity of your test assertions.

Output:

{% hint style="success" %}
00:00 +1: All tests passed!
{% endhint %}

This test also passes because `value` is both not null and greater than 0.

***

### Real-World Example: Testing a User Authentication Service

Let's explore a more realistic use case involving user authentication. In this example, we simulate an external authentication service using a class called `AuthService`, and test another class, `UserService`, that depends on it.

#### `AuthService` – Simulating External Logic

```dart
class AuthService {
  Future<bool> authenticate(String username, String password) async {
    // Imagine this calls an external authentication API
    return username == 'admin' && password == 'password';
  }
}
```

The `AuthService` class mimics a backend API call. It contains a method `authenticate` which checks if the username and password match hardcoded values. In a real-world scenario, this method would send HTTP requests to a server.

#### `UserService` – Business Logic Layer

```dart
class UserService {
  final AuthService authService;

  UserService(this.authService);

  Future<String> login(String username, String password) async {
    bool isAuthenticated = await authService.authenticate(username, password);
    return isAuthenticated ? 'Login successful' : 'Login failed';
  }
}
```

The `UserService` class represents your app’s internal logic. It depends on `AuthService` to handle authentication but transforms the result into user-friendly messages. We pass `AuthService` into the constructor to make it easily replaceable with a mock during testing (a technique called _dependency injection_).

***

#### Testing `UserService` with `mockito`

To test `UserService` without hitting a real API, we use the `mockito` package to simulate (`mock`) the behavior of `AuthService`.

First, create a mock class:

```dart
class MockAuthService extends Mock implements AuthService {}
```

This tells `mockito` to use `MockAuthService` as a stand-in for `AuthService` in your tests.

***

Test 1: Successful Login

```dart
test('returns success message when authentication is successful', () async {
  // Arrange: Set up a mock AuthService
  final mockAuthService = MockAuthService();
  when(mockAuthService.authenticate('admin', 'password'))
      .thenAnswer((_) async => true);

  final userService = UserService(mockAuthService);

  // Act: Call the login method
  final result = await userService.login('admin', 'password');

  // Assert: Expect a success message
  expect(result, equals('Login successful'));
});
```

* `when(...).thenAnswer(...)` sets up the mock to return `true` when given specific input.
* The `login` method is then tested to ensure it returns `'Login successful'`.

***

Test 2: Failed Login

```dart
test('returns failure message when authentication fails', () async {
  // Arrange
  final mockAuthService = MockAuthService();
  when(mockAuthService.authenticate('user', 'wrongpassword'))
      .thenAnswer((_) async => false);

  final userService = UserService(mockAuthService);

  // Act
  final result = await userService.login('user', 'wrongpassword');

  // Assert
  expect(result, equals('Login failed'));
});
```

This test ensures that when incorrect credentials are provided, the mocked authentication call returns `false`, and `UserService.login` returns the appropriate failure message.

***

Output:

{% hint style="success" %}
00:00 +2: All tests passed!
{% endhint %}

Both tests pass because `MockAuthService` behaves exactly as configured.

***

### Why Mocking Is Important

Using mocks allows you to:

* Test code in isolation without relying on network calls.
* Simulate various scenarios (success, failure, exceptions) with minimal setup.
* Run tests quickly and reliably.

***

### Best Practices for Writing Unit Tests

* **Isolate Units:** Test individual units of code in isolation by mocking dependencies.
* **Use Descriptive Test Names:** Clearly describe the behavior being tested.
* **Keep Tests Focused:** Each test should verify a single behavior or outcome.
* **Avoid External Dependencies:** Do not rely on network calls or file systems in unit tests; mock them instead.
* **Maintain Test Coverage:** Aim for high test coverage to catch regressions early.

### Conclusion

Unit testing in Dart is a powerful tool to ensure the reliability and correctness of your code. By following the practices outlined in this guide, you can write effective unit tests that facilitate confident refactoring and robust application development.
