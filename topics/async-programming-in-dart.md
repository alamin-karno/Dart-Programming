# Async Programming in Dart

Asynchronous programming is essential for building responsive applications. In Dart, the concept of asynchronous execution is closely tied with `Future` and `Stream`. While `Future` is used for single event async operations, `Stream` is used when you expect a sequence of asynchronous events. Whether you’re reading files, handling user input, or working with APIs, understanding Streams is crucial for writing efficient Dart applications.

### What is a Stream?

A `Stream` represents a sequence of asynchronous data events. Unlike a `Future`, which completes once, a Stream can emit multiple values over time. This makes it perfect for handling data that arrives gradually like user inputs, sensor data, or API responses in chunks.

There are two main types of Streams:

#### 1. Single-subscription Streams

These allow only one listener at a time. They're ideal when you have a known, linear flow of data like reading a file or making an HTTP request that delivers a sequence of values.

**Example:**

```dart
final stream = Stream.fromIterable([1, 2, 3]);

stream.listen((value) => print('Received: \$value'));
```

Output:

```yaml
Received: 1
Received: 2
Received: 3
```

This stream emits the numbers 1, 2, and 3 in order. It supports only one listener. If you try to listen again, it will throw an error.

#### 2. Broadcast Streams

These allow multiple listeners to subscribe to the same stream. Broadcast streams are useful when you want multiple parts of your app to react to the same events, such as user interactions or state changes like tracking app lifecycle changes, sensor updates, or user interactions.

**Example:**

```dart
final controller = StreamController<int>.broadcast();

controller.stream.listen((val) => print('Listener A: \$val'));
controller.stream.listen((val) => print('Listener B: \$val'));

controller.sink.add(1);
controller.sink.add(2);
```

Output:

```yaml
Listener A: 1
Listener B: 1
Listener A: 2
Listener B: 2
```

Here, two listeners are added to a single stream. Both will receive the same events emitted by the controller.

Broadcast streams allow multiple listeners to subscribe to the same stream. They’re useful when you want **various parts of your app to respond to the same events**, such as:

* Listening to **upload/download progress**
* **Live chat or data feed updates**
* Tracking **sensor changes**
* App-wide notifications (e.g., user logged out)

#### Real-World Example: Upload Progress Using Broadcast Stream

Let’s say you have a file upload feature in your app, and both the UI progress bar and the logs section need to update simultaneously.

We’ll use a `StreamController<double>.broadcast()` so multiple listeners can listen to the same stream.

**Step 1: Create the controller (shared source of updates)**

```dart
final uploadProgressController = StreamController<double>.broadcast();
```

***

Step 2: Log progress (e.g., in a service or background part)

```dart
uploadProgressController.stream.listen((progress) {
  print('Log: Uploaded ${(progress * 100).toStringAsFixed(0)}%');
});
```

This runs separately from UI. It prints progress like:\
`Log: Uploaded 10%`, `Log: Uploaded 20%`, ...

***

Step 3: Update UI (inside a StatefulWidget)

```dart
class UploadWidget extends StatefulWidget {
  @override
  _UploadWidgetState createState() => _UploadWidgetState();
}

class _UploadWidgetState extends State<UploadWidget> {
  double _progress = 0.0;
  StreamSubscription<double>? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = uploadProgressController.stream.listen((progress) {
      setState(() {
        _progress = progress;
      });
    });
  }

  @override
  void dispose() {
    _subscription?.cancel(); // Always cancel to avoid memory leaks
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return LinearProgressIndicator(value: _progress);
  }
}
```

The UI listens separately to the same stream, but manages state using `setState`.

***

Step 4: Simulate file upload

```dart
void simulateUpload() async {
  for (int i = 1; i <= 10; i++) {
    await Future.delayed(Duration(milliseconds: 500));
    uploadProgressController.sink.add(i / 10); // Emits 0.1, 0.2 ... 1.0
  }
  await uploadProgressController.close(); // Close when done
}
```

Now both parts of the app receive progress updates **independently but simultaneously**. This is where `BroadcastStream` shines — **a single stream, multiple consumers.**

### Creating Streams in Dart

#### 1. Using `Stream.fromIterable()`

* **Why?** Turns an existing collection into a stream of events, emitting each element sequentially.
* **When?** You have a fixed list of data (e.g., a list of IDs, file paths, or static configuration) and want to process items asynchronously.

```dart
final stream = Stream.fromIterable(["user1", "user2", "user3"]);

stream.listen((user) => print('Processing user: $user'));
```

Output:

```yaml
Processing user: user1
Processing user: user2
Processing user: user3
```

This creates a stream that emits elements from a list one by one.

#### 2. Using `Stream.periodic()`

* **Why?** Emits events at a fixed interval, ideal for polling or timers.
* **When?** You need repeated actions over time, such as refreshing data every few seconds or updating a stopwatch UI.

```dart
final stream = Stream.periodic(Duration(seconds: 1), (count) => count).take(3);

stream.listen((tick) => print('Tick $tick'));
```

Output (1 second delay between each line):

```yaml
Tick: 0
Tick: 1
Tick: 2
```

This emits a new value (based on `count`) every second. Useful for timers or polling.

Example: Polling for Server Status

```dart
final statusStream = Stream.periodic(Duration(seconds: 5), (_) => fetchServerStatus());

statusStream.listen((status) {
  print("Server status: $status");
});

Future<String> fetchServerStatus() async {
  // Simulate fetching server status
  await Future.delayed(Duration(milliseconds: 500));
  return DateTime.now().second % 2 == 0 ? 'Online' : 'Offline';
}
```

This demonstrates how to **use `Stream.periodic` for polling server health**, something common in dashboards or admin panels.

#### 3. Async Generator Function with `async*`

* **Why?** Lets you yield events from within an async function, combining asynchronous delay or I/O with stream emission.
* **When?** You need to fetch or compute data incrementally—e.g., loading paginated API results one page at a time, or reading large files chunk by chunk.

```dart
Stream<String> fetchPages(int totalPages) async* {
  for (int i = 1; i <= totalPages; i++) {
    await Future.delayed(Duration(seconds: 1)); // simulate delay
    yield 'Fetched page $i';
  }
}

fetchPages(3).listen((page) => print('Received: $page'));
```

Hypothetical Output (if `fetchPageFromApi` returns "Page $i" after delay):

```yaml
Received page: Page 1
Received page: Page 2
Received page: Page 3
```

This is an asynchronous generator. It yields a value after each delay. Ideal for step-by-step processing with delays or awaiting async tasks.

### Custom Streams with StreamController

`StreamController` provides fine-grained control over when and how events are emitted. Use it when events originate from multiple sources or when you need broadcast capabilities.

* **Why?** To manually add, pause, resume, or close a stream; useful for event buses, user interactions, or combining multiple inputs.
* **When?** You have custom data flows, such as mixing user clicks and network events, or implementing global notification systems.

```dart
final controller = StreamController<String>();
controller.stream.listen((event) => print("Received: \$event"));

controller.sink.add("Hello");
controller.sink.add("World");
await controller.close();
```

Expected Output:

```yaml
Received: Hello
Received: World
```

This approach is helpful when you're manually handling events—such as user inputs or system data.

### Managing StreamSubscription

A `StreamSubscription` gives you control over how and when to listen, pause, resume, or cancel stream events.

```dart
final subscription = controller.stream.listen((data) => print(data));

// Pause
subscription.pause();

// Resume
subscription.resume();

// Cancel
subscription.cancel();
```

Use this to manage the lifecycle of your listeners, especially in Flutter where you might need to stop listening on widget disposal.

### Stream Transformations

#### `map`, `where`, `take`, `skip`

```dart
stream.map((e) => e * 2).where((e) => e > 5);
```

Transforms values by doubling them and filters those greater than 5.

#### `asyncMap`

```dart
stream.asyncMap((e) async => await fetchData(e));
```

Used when transforming stream data into another async value. Useful for network calls or async parsing.

#### `expand` and `asyncExpand`

```dart
stream.expand((e) => [e, e * 10]);
stream.asyncExpand((e) async* {
  yield e;
  yield await computeSomethingElse(e);
});
```

`expand` flattens items into multiple values; `asyncExpand` does this with async logic. Useful when each item needs to result in multiple outputs.

### Error Handling in Streams

```dart
stream.listen(
  (data) => print(data),
  onError: (err) => print('Error: \$err'),
  onDone: () => print('Stream closed'),
);
```

Always handle errors to prevent unhandled exceptions, especially in production. `onDone` lets you clean up resources.

### Stream Optimization Tips

#### 1. Debounce Rapid Inputs

Use `debounceTime` to prevent handling every keystroke (useful for search input).

**Before:**

```dart
searchController.stream.listen((query) {
  print('Searching for: \$query');
});
```

**After:**

```dart
searchController.stream
  .debounceTime(Duration(milliseconds: 300))
  .listen((query) {
    print('Searching for: \$query');
  });
```

This waits 300ms after the last input before emitting, reducing API calls.

#### 2. Eliminate Redundant Events

Use `distinct()` to ignore repeated consecutive values.

```dart
stream.distinct().listen((value) {
  print('Received: \$value');
});
```

This avoids processing the same value multiple times in a row.

#### 3. Cancel Unused Streams

Free up memory and processing by canceling listeners when they’re no longer needed.

```dart
final subscription = stream.listen(print);
// Later in code
await subscription.cancel();
```

#### 4. Always Close Controllers

Avoid memory leaks by closing controllers when done.

```dart
final controller = StreamController<int>();
controller.sink.add(1);
await controller.close();
```

### Real-World Example: Debounced Search in a Flutter App

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

In this example, we debounce the user input in a `TextField`. If the user types quickly, the stream emits only the final input after 500ms of inactivity.

### Conclusion

Streams are a powerful part of Dart’s asynchronous capabilities. Understanding how to create, consume, transform, and optimize them will make your apps faster and more maintainable. Whether you're tracking live uploads, syncing real-time data, or reacting to user events, mastering Streams can dramatically elevate your Dart and Flutter development.
