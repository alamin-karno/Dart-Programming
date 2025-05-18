# Dart’s Type System

In Dart, everything has a type, even when you’re not explicitly specifying one. As an intermediate developer, understanding how Dart handle types can help you write more predictable, safe, and efficient code. This post will dive into two flexible aspects of Dart’s type system: `dynamic` and `Object`.

***

### What is a Type System?

A **type system** defines how a programming language classifies values and expressions into types. It ensures that operations are used with compatible types, either at **compile-time** (static typing) or **runtime** (dynamic typing).

Dart uses a **sound static type system** by default. This means the compiler checks types as much as possible at compile time—but also allows escape hatches like `dynamic`.

***

### `dynamic`: The Escape Hatch

The `dynamic` type tells Dart: _“Trust me, I know what I’m doing.”_ It disables compile-time type checking for a variable.

#### When to Use

* Reading untyped JSON data (e.g. from REST APIs).
* Writing generic data transformation utilities.
* Prototyping fast, flexible logic.

#### When to Avoid

* In business logic or UI logic where type safety is critical.
* When performance matters—`dynamic` incurs runtime overhead.
* For shared/public APIs—users lose type safety.

#### Example: Handling dynamic API response

```dart
void processApiResponse(dynamic response) {
  if (response is Map<String, dynamic>) {
    print('Received a map with keys: ${response.keys}');
  } else {
    print('Unexpected response type');
  }
}

void main() {
  dynamic apiData = {'id': 1, 'name': 'Dart'};
  processApiResponse(apiData);
}
```

**Best Practice**: Always check the runtime type before using dynamic values.

***

### `Object`: The Common Supertype

In Dart, `Object` is the superclass of all non-nullable types. It is more type-safe than `dynamic`, and method access is restricted to what's defined in `Object`.

#### When to Use

* When you need to store values of unknown type **but** don’t need to call arbitrary methods.
* When designing APIs or utility functions that need to accept **any type** of input.

#### When to Avoid

* If you’ll need to call methods not defined on `Object` without casting.
* When you want complete flexibility (in which case `dynamic` might be better).

#### Example: Type-safe collection

```dart
void logValues(List<Object> values) {
  for (var value in values) {
    print('Type: ${value.runtimeType}, Value: $value');
  }
}

void main() {
  logValues(['hello', 42, 3.14, true]);
}
```

**Best Practice**: Use `Object` when accepting any type of value but want to maintain compile-time safety.

***

### `var`, `Object`, and `dynamic`: A Comparison

| Declaration | Type Checking  | Type Inference | Use Case              |
| ----------- | -------------- | -------------- | --------------------- |
| `var`       | Static         | Yes            | Known type, inferred  |
| `Object`    | Static         | No             | Hold any value safely |
| `dynamic`   | None (runtime) | No             | Max flexibility       |

```dart
var a = 'hello';     // Inferred as String
Object b = 'world';  // Must cast to use String methods
dynamic c = '!!!';   // Can call anything; unsafe
```

***

### Real-World Case Study: JSON Parsing

When working with APIs that return untyped JSON, Dart developers often use `dynamic` or `Object` during decoding.

#### Dynamic JSON (common but risky)

```dart
dynamic data = jsonDecode(responseBody);
print(data['name']); // runtime error if 'name' is missing
```

Better Approach Using `Object` and Type Checking

```dart
Object rawData = jsonDecode(responseBody);
if (rawData is Map<String, dynamic>) {
  print('Name: ${rawData['name']}');
}
```

***

### Best Practices Summary

| Practice                          | Recommendation                   |
| --------------------------------- | -------------------------------- |
| Use `dynamic` cautiously          | Only when type is truly unknown  |
| Prefer `Object` over `dynamic`    | For general-purpose input/output |
| Check types at runtime            | Especially when using `dynamic`  |
| Avoid exposing `dynamic` in APIs  | Consumers lose type safety       |
| Use type inference (`var`) wisely | Cleaner code, fewer bugs         |

***

### Final Thoughts

Dart gives you powerful tools like `dynamic` and `Object` to work flexibly with unknown or changing data. But with great power comes great responsibility. Use these tools intentionally:

* Choose `Object` for safety.
* Use `dynamic` when flexibility is worth the tradeoff.
* Combine both with runtime type checks for robust apps.
