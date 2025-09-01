# 🚀 Flutter API Response Handling System (with Riverpod)

هذا النظام يوفر طريقة **احترافية وموحدة** للتعامل مع استجابات الـ API في تطبيقات Flutter باستخدام **Riverpod + StateNotifier**.
يساعدك على:

* ✅ إظهار رسالة نجاح (SnackBar).
* ❌ إظهار رسالة خطأ (SnackBar).
* 🌐 التعامل مع أخطاء الإنترنت (Dialog مع زر إعادة المحاولة).
* 📦 وضع كل شيء في مكان واحد وقابل لإعادة الاستخدام.

---

## 📂 هيكلة المشروع

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

## ⚙️ كيفية الاستخدام

### 1️⃣ تعريف موديل موحد للنتائج

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

### 2️⃣ الهاندلر المركزي

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

### 3️⃣ الإكستنشن لتبسيط الاستخدام

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

### 4️⃣ مثال StateNotifier

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

### 5️⃣ مثال في الواجهة

```dart
// features/data/data_screen.dart
class DataScreen extends ConsumerWidget {
  const DataScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(dataProvider);

    // 🪄 استدعاء السيستم مباشرة
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

## ✨ المزايا

* لا تحتاج تكرار كود الـ SnackBar/Dialog.
* مناسب لأي `StateNotifier` أو `FutureProvider`.
* يدعم retry تلقائي عند فقدان الاتصال.

---

## 📝 ملاحظات

* بإمكانك تخصيص رسائل النجاح/الفشل بسهولة من خلال الـ `ApiResult`.
* يمكن تطويره لاحقًا ليدعم أنواع أخرى من الأخطاء (Unauthorized, Forbidden...).

```}
```
