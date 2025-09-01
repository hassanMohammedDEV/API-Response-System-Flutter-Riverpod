# 🚀 Flutter API Response Handling System (with Riverpod)

This system provides a **professional and unified** way to handle API responses in Flutter applications using **Riverpod + StateNotifier**.
It helps you to:

* ✅ Show success messages (SnackBar).
* ❌ Show error messages (SnackBar).
* 🌐 Handle network errors (Dialog with Retry button).
* 📦 Keep everything centralized and reusable.

---

## 📂 Project Structure

```
lib/
├── core/
│   ├── api_result.dart        # Unified API result model
│   ├── api_handler.dart       # Centralized response handler (SnackBar/Dialog)
│   └── api_extensions.dart    # Extension for easy usage
├── features/
│   └── data/
│       ├── data_notifier.dart # Example StateNotifier with fake API
│       └── data_screen.dart   # Example usage in UI
```

---

## ⚙️ Usage

### 1️⃣ Define a unified API result model

```dart
// core/api_result.dart
sealed class ApiResult<T> {
  const ApiResult();
}

class ApiSuccess<T> extends ApiResult<T> {
  final String message;
  final T data;
  const ApiSuccess(this.data, {this.message = "Success"});
}

class ApiFailure<T> extends ApiResult<T> {
  final String message;
  const ApiFailure(this.message);
}

class ApiNetworkError<T> extends ApiResult<T> {
  final String message;
  const ApiNetworkError([this.message = "No internet connection"]);
}
```

---

### 2️⃣ Centralized handler

```dart
// core/api_handler.dart
class ApiHandler {
  static void handle<T>({
    required BuildContext context,
    required ApiResult<T> result,
    required VoidCallback onRetry,
  }) {
    if (result is ApiSuccess<T>) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(result.message)),
      );
    } else if (result is ApiFailure<T>) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(result.message)),
      );
    } else if (result is ApiNetworkError<T>) {
      showDialog(
        context: context,
        builder: (_) => AlertDialog(
          title: const Text("Connection Error"),
          content: Text(result.message),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.pop(context);
                onRetry();
              },
              child: const Text("Retry"),
            ),
          ],
        ),
      );
    }
  }
}
```

---

### 3️⃣ Extension for easy usage

```dart
// core/api_extensions.dart
extension ApiResultHandler<T> on AsyncValue<ApiResult<T>> {
  void handleApi(BuildContext context, {required VoidCallback onRetry}) {
    whenOrNull(
      data: (result) {
        ApiHandler.handle(context: context, result: result, onRetry: onRetry);
      },
    );
  }
}
```

---

### 4️⃣ Example StateNotifier

```dart
// features/data/data_notifier.dart
class DataNotifier extends StateNotifier<AsyncValue<ApiResult<String>>> {
  DataNotifier() : super(const AsyncValue.loading());

  Future<void> fetchData() async {
    try {
      await Future.delayed(const Duration(seconds: 1));
      state = AsyncValue.data(ApiSuccess("Hello World", message: "Loaded!"));
    } catch (_) {
      state = AsyncValue.data(ApiNetworkError());
    }
  }
}

final dataProvider =
    StateNotifierProvider<DataNotifier, AsyncValue<ApiResult<String>>>(
        (ref) => DataNotifier());
```

---

### 5️⃣ Example UI

```dart
// features/data/data_screen.dart
class DataScreen extends ConsumerWidget {
  const DataScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(dataProvider);

    // 🪄 Use the system directly
    state.handleApi(
      context,
      onRetry: () => ref.read(dataProvider.notifier).fetchData(),
    );

    return Scaffold(
      appBar: AppBar(title: const Text("API Handler Example")),
      body: Center(
        child: state.when(
          loading: () => const CircularProgressIndicator(),
          data: (result) {
            if (result case ApiSuccess(data: final value)) {
              return Text(value);
            }
            return const Text("No Data");
          },
          error: (_, __) => const Text("Unexpected Error"),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(dataProvider.notifier).fetchData(),
        child: const Icon(Icons.refresh),
      ),
    );
  }
}
```

---

## ✨ Features

* No need to repeat SnackBar/Dialog code.
* Works with any `StateNotifier` or `FutureProvider`.
* Supports automatic retry on network failure.

---

## 📝 Notes

* You can easily customize success/error messages through `ApiResult`.
* Can be extended later to handle other errors (Unauthorized, Forbidden, etc.).

```
```
