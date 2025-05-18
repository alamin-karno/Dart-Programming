# ডার্টে ইউনিট টেস্টিং

ইউনিট টেস্টিং প্রোগ্রামিংয়ের একটি গুরুত্বপূর্ণ অংশ, যা নিশ্চিত করে যে, আমাদের কোডের প্রতিটি ছোট ছোট অংশ ঠিকভাবে কাজ করছে কিনা। ডার্ট (Dart) প্রোগ্রামিংয়ে `test` প্যাকেজ ব্যবহার করে খুব সহজে ইউনিট টেস্ট লেখা ও run করা যায়। এই পর্বে আমরা দেখবো কিভাবে টেস্টিং সেটআপ করবো, সাধারণ ও অ্যাসিনক্রোনাস কোডের টেস্ট লিখবো, আর কিছু ভালো অভ্যাস মেনে চলবো যাতে টেস্টিং আরও ভালো হয়।

### টেস্টিং Environment সেটআপ করা

১। প্রথমেই তোমার প্রজেক্টে `test` প্যাকেজটা যোগ করতে হবে:

**pubspec.yaml** ফাইলে নিচের মতো করে `dev_dependencies` অংশে `test` প্যাকেজটা যোগ করতে হবে:

```yaml
dev_dependencies:
  test: ^1.21.0
```

তারপর টার্মিনালে গিয়ে নিচের কোডটি চালাতে হবে:

```bash
dart pub get
```

২। এবার প্রজেক্টের রুটে একটা `test` ফোল্ডার বানাতে হবে । টেস্ট ফাইলগুলোর নাম `_test.dart` দিয়ে শেষ করতে হবে , তাহলে টেস্ট রানার সেগুলো চিনতে পারবে।

***

### AAA প্যাটার্ন কি?

টেস্ট কোড লেখার জন্য AAA (Arrange, Act, Assert) প্যাটার্নটা অনেক জনপ্রিয়:

* **Arrange**: ডেটা বা অবজেক্ট যা দরকার তা প্রস্তুত করা
* **Act**: যে ফাংশন বা মেথড টেস্ট করছো তা চালানো
* **Assert**: রেজাল্টটা যেমন হওয়ার কথা ছিল, তা ঠিক আছে কিনা যাচাই করা

এই নিয়মে লেখা টেস্ট পড়তে ও বুঝতে অনেক সহজ হয়।

***

### একটা সাধারণ ইউনিট টেস্ট

ধরি তোমার একটা ফাংশন আছে যা দুইটা নাম্বার যোগ করে:

`lib/calculator.dart`

```dart
int add(int a, int b) {
  return a + b;
}
```

`test/calculator_test.dart`&#x20;

```dart
import 'package:test/test.dart';
import '../lib/calculator.dart';

void main() {
  test('দুইটি সংখ্যা যোগফল ঠিক দেয় কিনা', () {
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

এই টেস্ট চালালে আউটপুট হবে:

```makefile
00:00 +1: সব টেস্ট পাস করেছে!
```

***

### অ্যাসিনক্রোনাস কোড টেস্ট করা

যেমন ধরো তুমি একটা ডেটা লোড করার মতো ফাংশন বানিয়েছো:

`lib/data_fetcher.dart`&#x20;

```dart
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Data loaded';
}
```

`test/data_fetcher_test.dart`&#x20;

```dart
import 'package:test/test.dart';
import '../lib/data_fetcher.dart';

void main() {
  test('fetchData ঠিকভাবে রিটার্ন করে কিনা', () async {
    // Act
    String result = await fetchData();

    // Assert
    expect(result, equals('Data loaded'));
  });
}
```

এটা চালালে এক সেকেন্ড দেরিতে টেস্টটা পাস করবে:

```makefile
00:01 +1: সব টেস্ট পাস করেছে!
```

***

### আরও এক্সপ্রেসিভ টেস্ট লেখার জন্য Matcher ব্যবহার

`test` প্যাকেজে অনেক ধরণের Matcher আছে, যেগুলো দিয়ে assertion আরও পরিষ্কারভাবে লেখা যায়:

```dart
test('ভ্যালু নাল না এবং শূন্যের চেয়ে বড় কিনা', () {
  int value = 10;

  expect(value, isNotNull);
  expect(value, greaterThan(0));
});
```

আউটপুট:

```makefile
00:00 +1: সব টেস্ট পাস করেছে!
```

***

### উদাহরণ: ইউজার লগইন সার্ভিস টেস্ট করা

ধরি, তোমার একটা AuthService আছে, যেটা ইউজারনেম ও পাসওয়ার্ড যাচাই করে:

```dart
class AuthService {
  Future<bool> authenticate(String username, String password) async {
    return username == 'admin' && password == 'password';
  }
}
```

এখন `UserService` বানাই, যেটা AuthService ব্যবহার করে:

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

এখন, আমরা `mockito` প্যাকেজ ব্যবহার করে `AuthService`-কে মক করবো যাতে আসল সার্ভারে কল না দিতে হয়।

```dart
class MockAuthService extends Mock implements AuthService {}
```

টেস্ট ১: সফল লগইন

```dart
test('লগইন সফল হলে success মেসেজ দেয়', () async {
  final mockAuthService = MockAuthService();
  when(mockAuthService.authenticate('admin', 'password'))
      .thenAnswer((_) async => true);

  final userService = UserService(mockAuthService);

  final result = await userService.login('admin', 'password');

  expect(result, equals('Login successful'));
});
```

টেস্ট ২: ভুল পাসওয়ার্ড

```dart
test('লগইন ব্যর্থ হলে failed মেসেজ দেয়', () async {
  final mockAuthService = MockAuthService();
  when(mockAuthService.authenticate('user', 'wrongpassword'))
      .thenAnswer((_) async => false);

  final userService = UserService(mockAuthService);

  final result = await userService.login('user', 'wrongpassword');

  expect(result, equals('Login failed'));
});
```

আউটপুট:

```makefile
00:00 +2: সব টেস্ট পাস করেছে!
```

***

### মক ব্যবহার কেন জরুরি?

* টেস্টে নেটওয়ার্ক কল লাগে না
* যেকোনো পরিস্থিতি (সফল, ব্যর্থ, এক্সেপশন) সহজে সিমুলেট করা যায়
* টেস্ট চালানো হয় দ্রুত ও নির্ভরযোগ্যভাবে

***

### ভালো টেস্ট লেখার কিছু পরামর্শ

* **Isolate Units**: প্রতিটি ফাংশনের টেস্ট আলাদাভাবে করা
* **বুঝতে সুবিধা হয় এমন নাম দেয়া**: টেস্টের নাম যেন বুঝায় কি যাচাই করা হচ্ছে
* **এক টেস্টে একটাই কাজ করো**: টেস্ট যেন একটাই কাজ যাচাই করে
* **বাইরের কিছুর ওপর নির্ভর না করা**: API বা ফাইল সিস্টেমের ওপর নির্ভর না করে Mock ব্যবহার করা
* **টেস্ট কাভারেজ ঠিক রাখা**: যতটা সম্ভব বেশি কোড টেস্ট কভার করো

***

### উপসংহার

ডার্টে ইউনিট টেস্টিং কোডকে reliability আর রিফ্যাক্টরিং-বান্ধব করে তোলে। এইখানের শেখা নিয়মগুলো মেনে চললে, তুমি এমন টেস্ট লিখতে পারবে যা ভবিষ্যতের যেকোনো সমস্যা আগে থেকেই ধরে ফেলতে পারবে।
