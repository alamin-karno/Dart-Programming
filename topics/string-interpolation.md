---
description: >-
  এই পর্বে আমরা Dart Programming Language এর গুরুত্বপূর্ণ একটি Topic নিয়ে জানব
  সেটি হল String Interpolation. এইটা শিখার আগে আমরা Literals এবং Strings
  Literals নিয়ে জানব ।
---

# String Interpolation

### Literals

মূলত Programing এর ভাষায় literals বলতে বুঝায় কোন Variable এর মধ্যে আমরা যেই value গুলা assign করি তাদের । নিচে কিছু উদাহরণ দেয়া হলঃ

```dart
// Literals Example
  var isOK = true;
  var x = 5;
  var n = "Md. Al-Amin";
  var d = 1.2;
```

### Various way to define String Literals

```dart
  String s1 = 'Single String';
  String s2 = "Double String";
  
  String s3 = 'I don\'t want to eat.'; 
  
  String s4 = "I don't want to eat.";
  
  String s5 = 'This is a long String to check string literals.'
    'This is a long String to check string literals.'
    'This is a long String to check string literals.';
```

### String Interpolation

মূলত কিভাবে String এর Interpolation হয় তা আমরা শিখব নিচের উদাহরণ থেকে ।&#x20;

```dart
  // String Interpolation Examples
  
  String name = 'Md. Al-Amin';
  String message = 'My name is $name';
  
  print(message);
  
  print('My name is $name');
  
  print('Total Character: ${name.length}');
  
  var a = 5;
  var b = 10;
  
  print('The sum of $a and $b is: ${a+b}');
  
  
  print('Multiplication of $a and $b is = ${a * b}');
```

{% hint style="success" %}
My name is Md. Al-Amin&#x20;

My name is Md. Al-Amin&#x20;

Total Character: 11&#x20;

The sum of 5 and 10 is: 15&#x20;

Multiplication of 5 and 10 is = 50
{% endhint %}

### String Interpolation নিয়ে বিস্তারিত ভিডিও নিচে দেয়া হলঃ

{% embed url="https://youtu.be/b2mjeTzQw28" %}
Literals & String Interpolations in Dart | 06 | Dart Tutorial for Flutter Application Development
{% endembed %}

{% hint style="info" %}
অবশ্যই আপনার মুল্যবান মন্তব্য জানাতে ভুলবেন নাহ ।&#x20;
{% endhint %}
