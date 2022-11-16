---
description: এইখানে আমরা Dart Programing এর Data Types নিয়ে জানব ।
---

# Data Types

### int - data types example

```dart
void main() {
  
  int age = 13;
  
  print(age);
  
}
```

{% hint style="success" %}
13
{% endhint %}

### String - data types example

```dart
void main() {
  
  String name = "Alamin";
  
  print(name);
  
}
```

{% hint style="success" %}
Alamin
{% endhint %}

### int, double, String, boolean - data types example

```dart
void main() {
  
  String name;
  int num;
  bool statement;
  double number;
  
  
  name = "Alamin";
  num = 5;
  statemnet = true;
  number = 5.5;
  
  print(name);
  print(num);
  print(statement);
  print(number);
}
```

{% hint style="success" %}
Alamin

5

true

5.5
{% endhint %}

### কিভাবে Data Types বের করব ?

```dart
void main() {
  
  var name;
  var num;
  var statement;
  var number;
  
  
  name = "Alamin";
  num = 5;
  statemnet = true;
  number = 5.5;
  
  print(name.runtimeType);
  print(num.runtimeType);
  print(statement.runtimeType);
  print(number.runtimeType);
}
```

{% hint style="success" %}
String&#x20;

int&#x20;

bool&#x20;

double
{% endhint %}

### Data Types নিয়ে বিস্তারিত ভিডিও ?

{% embed url="https://youtu.be/DRWSC5oyKLs" %}

{% hint style="info" %}
**অবশ্যই ভিডিও টি দেখবেন ভাল করে বুঝার জন্য ।**
{% endhint %}
