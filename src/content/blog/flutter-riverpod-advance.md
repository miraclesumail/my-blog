---
title: 'flutter-riverpod-advance'
description: 'flutter进阶'
pubDate: 'Jan 18 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }

  p {
    margin: 0 !important;
  }

  li {
     font-size: 16px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    margin-bottom: 8px;
  }
</style>

1. 🤖 [入门](#basic)
2. ⚙️ [类和状态](#simple)
3. 🔋 [异步](#future)
4. 🤸 [升级版autodispose](#dispose)

#### <a name="basic">🤖 一个最简单的demo</a>

<div class="small-code">

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// --- 第一步：定义数据源 (Provider) ---
// StateProvider 适合简单的状态，比如 counter
final counterProvider = StateProvider<int>((ref) => 0);

void main() {
  runApp(
    // --- 第二步：注入作用域 ---
    // 必须用 ProviderScope 包裹根组件，否则 ref 无法找到数据
    const ProviderScope(child: MyApp()),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: CounterPage(),
    );
  }
}

// --- 第三步：使用 ConsumerWidget ---
// 注意这里继承的是 ConsumerWidget，而不是 StatelessWidget
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 使用 ref.watch 监听数据。当 counterProvider 改变时，这里会自动重新 build
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Riverpod Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('你点击按钮的次数：'),
            Text(
              '$count',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ))
  }
}

```

</div>

#### <a name="simple">⚙️ 类和状态</a>

<div class="small-code">

```dart
// 1. 定义状态
class ProductsNotifier extends Notifier<List<String>> {
  @override
  List<String> build() => ['Apple', 'Banana']; // 初始数据

  void addProduct(String name) {
    // 注意：Riverpod 提倡“不可变性”，所以要用解构赋值创建一个新 List
    state = [...state, name];
  }

  void removeProduct(String name) {
  // 逻辑：保留所有“不等于该名称”的水果
  // 这会创建一个全新的 List 实例
    state = state.where((item) => item != name).toList();
  }

  void addMultipleProducts(List<String> newFruits) {
    // 逻辑：将旧状态的所有元素和新列表的所有元素合并到一个新 List 中
    state = [...state, ...newFruits];
 }
}

// 2. 暴露 Provider
final productsProvider = NotifierProvider<ProductsNotifier, List<String>>(ProductsNotifier.new);

// 3. UI 中调用
class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
     final productLogic = ref.read(productsProvider.notifier)
     productLogic.addProduct('Orange');
     productLogic.removeProduct('Orange');
  }
}

```

</div>

#### <a name="future">🔋 异步数据操作</a>

<div class="small-code">

```dart
// 假设我们有一个简单的模型
class Product {
  final String id;
  final String name;
  final double price;
  int stock;

  Product({required this.id, required this.name, required this.price, required this.stock});

  // 用于不可变更新
  Product copyWith({int? stock, String? name, double? price}) {
    return Product(
      id: id,
      name: name ?? this.name,
      price: price ?? this.price,
      stock: stock ?? this.stock,
    );
  }
}
```

<p>异步provider</p>

```dart
class ProductsNotifier extends AsyncNotifier<List<Product>> {
  // 原始数据备份，用于搜索恢复
  List<Product> _allProducts = [];

  @ovveride
  Future<List<Product>> build() async {
    // 初始加载数据
    _allProducts = await _fetchFromApi();
    return _allProducts;
  }

  // 模拟 API 请求
  Future<List<Product>> _fetchFromApi() async {
    await Future.delayed(const Duration(seconds: 1));
    return [
      Product(id: '1', name: 'iPhone 15', price: 5999, stock: 10),
      Product(id: '2', name: 'MacBook Pro', price: 12999, stock: 5),
      Product(id: '3', name: 'iPad Air', price: 4799, stock: 8),
      Product(id: '4', name: 'AirPods', price: 999, stock: 20),
    ];
  }

  Future<Void> loadMore() async {
    // 如果已经在加载了，直接返回防止重复请求
    if (state.isLoading) return;

    // 保留旧数据，并将状态设为 loading
    final previousState = state.value ?? [];
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      // 生成随机数据10条 自己模拟去写
      final newItems = await _fetchProductsFromApi(count: 10);
      return [...previousState, ...filteredNewItems];
    });
  }

  // --- 1. 搜索过滤逻辑 ---
  void filterByName(String query) {
    if (query.isEmpty) {
      state = AsyncValue.data(_allProducts);
      return;
    }

    // 在当前已加载的数据中过滤
    final filtered = _allProducts
        .where((p) => p.name.toLowerCase().contains(query.toLowerCase()))
        .toList();

    state = AsyncValue.data(filtered);
  }

  // 2. 复杂逻辑：局部修改（比如购买了一件，减库存）
  void purchase(String productId) {
    if (!state.hasValue) return;

    // Riverpod 的核心：通过创建新 List 来触发 UI 刷新（不可变性）
    state = AsyncValue.data([
      for (final p in state.value!)
        if (p.id == productId)
          p.copyWith(stock: p.stock - 1) // 只修改对应的那个对象
        else
          p,
    ]);
  }

}
// 定义 Provider
final productsProvider = AsyncNotifierProvider<ProductsNotifier, List<Product>>(ProductsNotifier.new);
```

<p>在ui中使用</p>

```dart
class ProductsList extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 获取异步状态
    final asyncProducts = ref.watch(productsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('高级商场')),
      body: asyncProducts.when(
        // 加载完成
        data: (products) => ListView.builder(
          itemCount: products.length,
          itemBuilder: (context, index) {
            final p = products[index];
            return ListTile(
              title: Text(p.name),
              subtitle: Text("库存: ${p.stock}"),
              trailing: ElevatedButton(
                onPressed: p.stock > 0
                  ? () => ref.read(productsProvider.notifier).purchase(p.id)
                  : null,
                child: const Text("购买"),
              ),
            );
          },
        ),
        // 加载中
        loading: () => const Center(child: CircularProgressIndicator()),
        // 加载出错
        error: (err, stack) => Center(child: Text("加载失败: $err")),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(productsProvider.notifier).loadMore(),
        child: const Icon(Icons.download),
      ),
    );
  }
}

```

</div>

#### <a name="dispose">🤸 升级版autodispose</a>

<div class="small-code">

<p>定义http service</p>

```dart
// http工具类似axios
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final dioProvider = Provider(
  (ref) => Dio(
    BaseOptions(
      baseUrl: 'https://jsonplaceholder.typicode.com', // 关键：模拟一个真实的浏览器标识
      headers: {
        'user-agent':
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
        'Accept': 'application/json',
      },
    ),
  ),
);

final postServiceProvider = Provider((ref) {
  final dio = ref.watch(dioProvider);
  return PostService(dio);
});

```

<p>service和model定义</p>

```dart
class Post {
  final int id;
  final String title;
  final String body;

  Post({required this.id, required this.title, required this.body});

  // 核心：将 Map 转为对象 (反序列化)
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }
}

class PostService {
  final Dio _dio;

  PostService(this._dio);

  Future<Post> getPostById(int id) async {
    print(id);
    try {
      final response = await _dio.get(('/posts/$id'));
      print(response);
      return Post.fromJson(response.data);
    } on DioException catch (e) {
      // 专门捕获 Dio 异常
      if (e.response?.statusCode == 403) {
        print("服务器拒绝访问：可能是 API 频率限制或缺少 Header");
      }
      // 重新抛出一个业务层能听懂的错误
      throw _handleDioError(e);
    } catch (e) {
      // 捕获其他非网络错误（如 JSON 解析失败）
      throw Exception("未知错误: $e");
    }
  }
}

// 封装一个错误处理器
String _handleDioError(DioException e) {
  switch (e.type) {
    case DioExceptionType.connectionTimeout:
      return "网络连接超时";
    case DioExceptionType.badResponse:
      return "服务器返回错误: ${e.response?.statusCode}";
    default:
      return "网络请求失败，请稍后再试";
  }
}

```

<p>最后定义provider</p>

```dart
// [AutoDispose] [Family] [Async] [Notifier]
class PostController extends AutoDisposeFamilyAsyncNotifier<Post, int> {
  @override
  FutureOr<Post> build(int arg) {
    return _fetch(arg);
  }

  Future<Post> _fetch(int id) async {
    final service = ref.read(postServiceProvider);
    final data = await service.getPostById((id));
    return data;
  }

  Future<void> refreshPost() async {
    state = const AsyncLoading();
    await Future.delayed(const Duration(seconds: 2));
    state = await AsyncValue.guard(() => _fetch(arg));
  }
}

// 注意使用 AsyncNotifierProvider.autoDispose.family
final postControllerProvider = AsyncNotifierProvider.autoDispose.family<PostController, Post, int>(PostController.new)

```

<p>ui中使用</p>

```dart

class PostDetailNew extends ConsumerWidget {
  final int postId;

  const PostDetailNew({super.key, required this.postId});

  Widget _buildPostContent(Post post, PostController postLogic) {

  }

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final postAsync = ref.watch(postControllerProvider(postId));
    final postLogic = ref.read(postControllerProvider(postId).notifier);

    return AsyncValueWidget(
      value: postAsync,
      data: (post) => _buildPostContent(post, postLogic),
    );
  }
}  

// 异步组价渲染
class AsyncValueWidget<T> extends StatelessWidget {
  final AsyncValue<T> value;
  // 一个传参为T返回Widget的函数data
  final Widget Function(T data) data;

  const AsyncValueWidget({super.key, required this.value, required this.data});

  @override
  Widget build(BuildContext context) {
    return value.when(
      data: data,
      error: (e, stack) => Center(
        child: Column(
          children: [
            Text('出错了：$e'),
            ElevatedButton(onPressed: () {}, child: const Text('重试')),
          ],
        ),
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
    );
  }
}

```

</div>
