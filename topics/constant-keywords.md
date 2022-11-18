---
description: >-
  আজকে আমরা Dart Programing এর মধ্যে কিভাবে Constant Value ব্যবহার করতে হয় তা
  শিখবো ।
---

# Constant Keywords

মূলত Constant Variable ব্যবহার করার জন্য Dart Programming Language এর মধ্যে দুইটি keyword রয়েছে । '**final**' এবং '**const**' । আমরা যদি কোন এমন কোন Variable ব্যবহার করি যার value কখনো চেঞ্জ হবে না তখন আমরা চাইলেই এই keyword ব্যবহার করতে পারি । এখন প্রশ্ন হল কখন এদের ব্যবহার করব ? এবং কি ভাবে ব্যবহার করতে হয় ? তাছাড়া দুইটার মধ্যে পার্থক্য কি ? তাই আজকে আমরা শিখবো ।&#x20;

### 'final' Keyword

'**final**' keyword মূলত Initialize হয় যখন আমরা সেই value access করব । তারমানে আমরা যদি কোন value এর type এর আগে final keyword ব্যবহার করি এবং সেই variable যতক্ষণ পর্যন্ত ব্যবহার না করব , ততক্ষণ পর্যন্ত এটি ম্যামোরিতে জায়গা নিবে না ।&#x20;

**Example:**

```dart
void main() {
  
  final city = 'Dhaka';  // datatype ছাড়া final keyword এর ব্যবহার
  
  final String country = 'Bangladesh'; 
   
}
```

'**final**' keyword dclear এর সময় যদি আমরা variable এর value assign করে দেই তাহলে আমরা চাইলে variable এর datatype declear না করেও variable declear করতে পারি ।&#x20;

### 'const' Keyword

'**const**' keyword অনেকটাই **final** এর মতই কাজ করে শুধুমাত্র এইটা initilize হয় compile time এ । মানে আমরা যখনি কোন একটা program compile করি তখনি '**const**' variable গুলো initialize হয়ে যায় , যার ফলে এই variable ব্যবহার করি না আর করি এই variable ম্যামরিতে যায়গা নিয়ে রাখে ।&#x20;

**Example:**

```dart
void main() {

 
   const PI = 3.1416;      // datatype ছাড়া const keyword এর ব্যবহার
  
  const double gravity = 9.82;
  
}
```

'**const**' keyword dclear এর সময় যদি আমরা variable এর value assign করে দেই তাহলে আমরা চাইলে variable এর datatype declear না করেও variable declear করতে পারি ।&#x20;

### 'static' Keyword

Class level instance varibale এর ক্ষেত্রে অবশ্যই মনে রাখতে হবে যে শুধুমাত্র '**final**' keyword ব্যবহার করা যায় । '**const**' ব্যবহার করা যায় না । যদি '**const** ব্যবহার  করতে হয় তাহলে **const** এর আগে **static** keyword ব্যবহার করতে হয় ।&#x20;

**Example:**

```dart
class Squre{
  
  final color = 'Green';
  static const PI = 3.1416;   // static ব্যবহার না করলে Error আসত
  
}
```

