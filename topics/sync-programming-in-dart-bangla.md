---
icon: group-arrows-rotate
---

# ডার্টে অ্যাসিনক্রোনাস প্রোগ্রামিং

রেসপন্সিভ অ্যাপ তৈরি করতে অ্যাসিনক্রোনাস প্রোগ্রামিং খুব গুরুত্বপূর্ণ। Dart-এ অ্যাসিনক্রোনাস `Future` এবং `Stream` এর মাধ্যমে execution হয়ে থাকে। `Future` সাধারণত একবারের জন্য কোনো অ্যাসিনক্রোনাস অপারেশনের ফলাফল দেয়, কিন্তু `Stream` ব্যবহার করা হয় যখন একের পর এক অনেকগুলো অ্যাসিনক্রোনাস ইভেন্টের অপেক্ষা করার দরকার পরে।

ফাইল read করা, ইউজার ইনপুট হ্যান্ডেল করা কিংবা API থেকে ডেটা নেওয়ার মতো কাজগুলোর জন্য Stream সম্পর্কে ভালোভাবে বোঝা Dart-এ দক্ষ অ্যাপ বানাতে খুবই প্রয়োজন।

***

### Stream কী?

`Stream` হলো অ্যাসিনক্রোনাস ইভেন্টগুলোর একটি ধারাবাহিক সিকোয়েন্স। `Future` যেখানে একবারের জন্য কোনো রেজাল্ট দেয়, সেখানে `Stream` একাধিক মান সময়ের সাথে সাথে দিতে পারে। তাই এটা সেই পরিস্থিতিতে খুবই উপকারী, যেখানে ডেটা আসতে থাকে ধাপে ধাপে — যেমন ইউজারের ইনপুট, সেন্সর ডেটা বা বড় আকারের API রেসপন্স।

***

### Stream-এর ধরন

Dart-এ মূলত দুটি ধরণের Stream রয়েছে:

**১. Single-subscription Stream**

এই ধরণের Stream-এ একবারে একজন লিসেনার (listener) যুক্ত হতে পারে। সাধারণত যখন আপনি জানেন যে একটি নির্দিষ্ট ধাপে ডেটা আসবে — যেমন ফাইল read করা বা সিকোয়েন্সভিত্তিক HTTP রিকোয়েস্ট — তখন এটি ব্যবহার করা হয়।

**উদাহরণ:**

```dart
final stream = Stream.fromIterable([1, 2, 3]);

stream.listen((value) => print('প্রাপ্ত ডেটা: $value'));
```

আউটপুট:

```yaml
প্রাপ্ত ডেটা: 1  
প্রাপ্ত ডেটা: 2  
প্রাপ্ত ডেটা: 3  
```

এখানে Stream একে একে ১, ২, ৩ পাঠাচ্ছে। এটি শুধু একবারই লিসেন করতে পারবে। আবার চেষ্টা করলে Error দিবে।

***

২. Broadcast Stream

Broadcast Stream-এ একাধিক লিসেনার যুক্ত হতে পারে। যখন আপনার অ্যাপের বিভিন্ন অংশকে একই ইভেন্টে react করতে হয়, তখন এটি খুব কার্যকর — যেমন সেন্সর data আপডেট, ইউজার ইন্টারঅ্যাকশন বা অ্যাপ লাইফসাইকেল ট্র্যাক করা।

**উদাহরণ:**

```dart
final controller = StreamController<int>.broadcast();

controller.stream.listen((val) => print('লিসেনার A: $val'));
controller.stream.listen((val) => print('লিসেনার B: $val'));

controller.sink.add(1);
controller.sink.add(2);
```

আউটপুট:

```yaml
লিসেনার A: 1  
লিসেনার B: 1  
লিসেনার A: 2  
লিসেনার B: 2  
```

এখানে একটি ইভেন্ট একাধিক লিসেনার পাচ্ছে। এটা একই ডেটা অ্যাপের একাধিক জায়গায় পাঠাতে খুব কার্যকর।

***

### উদাহরণ: ফাইল আপলোড ট্র্যাকার

ধরুন আপনার অ্যাপে একটি ফাইল আপলোড ফিচার আছে এবং আপলোড প্রগ্রেস UI ও লগ সেকশন — দুই জায়গায়ই আপডেট পাঠাতে হবে।

```dart
final uploadProgressController = StreamController<double>.broadcast();

// UI আপডেট
uploadProgressController.stream.listen((progress) {
  print('প্রগ্রেস বার আপডেট: ${progress * 100}%');
});

// লগ আপডেট
uploadProgressController.stream.listen((progress) {
  print('লগ: আপলোড হয়েছে ${(progress * 100).toStringAsFixed(0)}%');
});

void simulateUpload() async {
  for (int i = 1; i <= 10; i++) {
    await Future.delayed(Duration(milliseconds: 500));
    uploadProgressController.sink.add(i / 10);
  }
  await uploadProgressController.close();
}

simulateUpload();
```

আউটপুট:

```yaml
প্রগ্রেস বার আপডেট: 10.0%  
লগ: আপলোড হয়েছে 10%  
...  
প্রগ্রেস বার আপডেট: 100.0%  
লগ: আপলোড হয়েছে 100%  
```

এইভাবে Broadcast Stream ব্যবহার করে একই ডেটা অ্যাপের একাধিক অংশে পাঠানো যায়।

***

### Dart-এ Stream তৈরি করার পদ্ধতি

#### ১. `Stream.fromIterable()` ব্যবহার করে

**কেন ব্যবহার করবেন?**\
একটি লিস্ট বা কালেকশনকে ধাপে ধাপে স্ট্রিমের মাধ্যমে পাঠাতে চাইলে।

**কখন ব্যবহার করবেন?**\
যখন আপনার কাছে পূর্বনির্ধারিত কিছু ডেটা আছে (যেমন ইউজার আইডির তালিকা, ফাইলের path, কনফিগারেশন ইত্যাদি) এবং সেগুলো অ্যাসিনক্রোনাসভাবে process করতে চান।

**উদাহরণ:**

```dart
final stream = Stream.fromIterable(["user1", "user2", "user3"]);

stream.listen((user) => print('ইউজার process হচ্ছে: $user'));
```

আউটপুট:

```yaml
ইউজার process হচ্ছে: user1  
ইউজার process হচ্ছে: user2  
ইউজার process হচ্ছে: user3  
```

***

#### ২. `Stream.periodic()` ব্যবহার করে

**কেন ব্যবহার করবেন?**\
নির্দিষ্ট সময় পরপর একটি ইভেন্ট ট্রিগার করতে চাইলে এটি আদর্শ। যেমন টাইমার বা সার্ভার পোলিং।

**কখন ব্যবহার করবেন?**\
যখন আপনার দরকার নির্দিষ্ট সময় পরপর অ্যাকশন চালানো — যেমন প্রতি ৫ সেকেন্ডে সার্ভারের অবস্থা দেখা বা প্রতি ১ সেকেন্ডে টাইমার আপডেট করা।

**উদাহরণ:**

```dart
final stream = Stream.periodic(Duration(seconds: 1), (count) => count).take(3);

stream.listen((tick) => print('টিক: $tick'));
```

আউটপুট (প্রতি সেকেন্ডে একবার):

```yaml
টিক: 0  
টিক: 1  
টিক: 2  
```

সার্ভার স্ট্যাটাস চেকের উদাহরণ:

```dart
final statusStream = Stream.periodic(Duration(seconds: 5), (_) => fetchServerStatus());

statusStream.listen((status) {
  print("সার্ভারের অবস্থা: $status");
});

Future<String> fetchServerStatus() async {
  await Future.delayed(Duration(milliseconds: 500));
  return DateTime.now().second % 2 == 0 ? 'অনলাইনে' : 'অফলাইনে';
}
```

***

#### ৩. `async*` দিয়ে অ্যাসিনক্রোনাস জেনারেটর ফাংশন

**কেন ব্যবহার করবেন?**\
যখন প্রতিটি ইভেন্টের আগে একটু অপেক্ষা করতে হয়, বা ইভেন্ট তৈরি হয় কোনো অ্যাসিনক্রোনাস ফাংশনের মাধ্যমে।

**কখন ব্যবহার করবেন?**\
যখন আপনি ধাপে ধাপে ডেটা আনছেন — যেমন API থেকে পেজ বাই পেজ রেসপন্স আনছেন বা বড় ফাইল ভাগে ভাগে পড়ছেন।

**উদাহরণ:**

```dart
Stream<String> fetchPages(int totalPages) async* {
  for (int i = 1; i <= totalPages; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield 'পৃষ্ঠা $i ডাউনলোড হয়েছে';
  }
}

fetchPages(3).listen((page) => print('প্রাপ্ত: $page'));
```

আউটপুট:

```yaml
প্রাপ্ত: পৃষ্ঠা 1 ডাউনলোড হয়েছে  
প্রাপ্ত: পৃষ্ঠা 2 ডাউনলোড হয়েছে  
প্রাপ্ত: পৃষ্ঠা 3 ডাউনলোড হয়েছে  
```

***

### StreamController দিয়ে নিজের মতো করে Custom Stream তৈরি করা

`StreamController` আপনাকে স্ট্রিম কবে শুরু হবে, কী পাঠাবে, কখন বন্ধ হবে ইত্যাদি ব্যাপারে পূর্ণ নিয়ন্ত্রণ দেয়।

**কেন ব্যবহার করবেন?**\
যখন আপনি নিজে থেকে ইভেন্ট ট্রিগার করতে চান — যেমন ব্যবহারকারী ক্লিক করলে, অথবা একাধিক ডেটা সোর্স থেকে ইভেন্ট আসলে।

**উদাহরণ:**

```dart
final controller = StreamController<String>();

controller.stream.listen((event) => print("প্রাপ্ত: $event"));

controller.sink.add("হ্যালো");
controller.sink.add("ওয়ার্ল্ড");
await controller.close();
```

আউটপুট:

```yaml
প্রাপ্ত: হ্যালো  
প্রাপ্ত: ওয়ার্ল্ড  
```

***

### StreamSubscription ম্যানেজ করা

StreamSubscription দিয়ে আপনি একটি স্ট্রিমে লিসেন করা, পজ করা, রিজিউম বা বাতিল করতে পারেন।

```dart
final subscription = controller.stream.listen((data) => print(data));

// থামানো
subscription.pause();

// আবার চালু করা
subscription.resume();

// বাতিল করা
subscription.cancel();
```

Flutter-এ Widget dispose হওয়ার সময় StreamSubscription বন্ধ করা খুব জরুরি।

***

### স্ট্রিম ট্রান্সফরমেশন

#### `map`, `where`, `take`, `skip`:

```dart
stream.map((e) => e * 2).where((e) => e > 5);
```

একটি স্ট্রিমের প্রতিটি ভ্যালু দ্বিগুণ করে এবং যেগুলো ৫-এর বেশি, সেগুলো ধরে।

`asyncMap`:

```dart
stream.asyncMap((e) async => await fetchData(e));
```

প্রতিটি আইটেমকে অ্যাসিনক্রোনাস ভ্যালুতে রূপান্তর করে — যেমন নেটওয়ার্ক কল।

#### `expand` ও `asyncExpand`:

```dart
stream.expand((e) => [e, e * 10]);

stream.asyncExpand((e) async* {
  yield e;
  yield await computeSomethingElse(e);
});
```

একটি আইটেম থেকে একাধিক আইটেম তৈরি করা — Sync ও Async উভয়ভাবেই।

***

### স্ট্রিমে এরর হ্যান্ডলিং

```dart
stream.listen(
  (data) => print(data),
  onError: (err) => print('এরর: $err'),
  onDone: () => print('স্ট্রিম শেষ'),
);
```

সবসময় `onError` ও `onDone` হ্যান্ডল করা উচিত, বিশেষ করে প্রোডাকশন অ্যাপে।

***

### Stream অপ্টিমাইজেশন টিপস

#### ১. দ্রুত ইনপুট ডিবাউন্স করা

আগের কোড:

```dart
searchController.stream.listen((query) {
  print('সার্চ করা হচ্ছে: $query');
});
```

ডিবাউন্স করার পর:

```dart
searchController.stream
  .debounceTime(Duration(milliseconds: 300))
  .listen((query) {
    print('সার্চ করা হচ্ছে: $query');
  });
```

একটি ইনপুটের পরে ৩০০ মিলিসেকেন্ড ইনঅ্যাকটিভ থাকবে এর পরেই পরের API কল হবে।

২. একই ডেটা বারবার এড়ানো

```dart
stream.distinct().listen((value) {
  print('প্রাপ্ত: $value');
});
```

একই মান পরপর আসলে একবারই প্রসেস করা হবে।

৩. প্রয়োজন শেষ হলে স্ট্রিম বন্ধ করা

```dart
final subscription = stream.listen(print);

// পরে
await subscription.cancel();
```

৪. সবসময় Controller বন্ধ করা

```dart
final controller = StreamController<int>();
controller.sink.add(1);
await controller.close();
```

না বন্ধ করলে মেমোরি লিক হতে পারে।

***

### বাস্তব উদাহরণ: Flutter অ্যাপে ডিবাউন্স সার্চ

```dart
final searchController = StreamController<String>.broadcast();

TextField(
  onChanged: (value) => searchController.sink.add(value),
);

searchController.stream
  .debounceTime(Duration(milliseconds: 500))
  .listen((searchQuery) {
    fetchResults(searchQuery);
  });
```

ব্যবহারকারী যত দ্রুত টাইপই করুক না কেন, ৫০০ মিলিসেকেন্ড থেমে তবেই সার্চ চালানো হবে।

***

### উপসংহার

`Stream` Dart-এর অ্যাসিনক্রোনাস প্রোগ্রামিংয়ের একটি শক্তিশালী দিক। সঠিকভাবে Stream তৈরি, ব্যবহার, রূপান্তর ও অপ্টিমাইজ করতে পারলে আপনার অ্যাপ আরও দ্রুততর, স্মার্ট ও user-friendly হবে। লাইভ আপলোড ট্র্যাকিং, রিয়েলটাইম ডেটা সিঙ্কিং, বা ইউজার ইভেন্টে রেসপন্স — সব ক্ষেত্রেই Stream দক্ষভাবে ব্যবহার করা আপনাকে এগিয়ে রাখবে।
