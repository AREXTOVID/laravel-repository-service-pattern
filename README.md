# 🧠 ليه نستخدم (Interface - Repository - Service) في Laravel؟

## ✅ الهدف:

نفصل المسؤوليات (Separation of Concerns) وده مهم جدًا في المشاريع المتوسطة والكبيرة.

- **Controller:** يستقبل الطلب ويتعامل مع الرد (Response).
- **Service:** يحتوي على منطق العمل (Business Logic).
- **Repository:** يتعامل مع قاعدة البيانات مباشرة.
- **Interface:** يحدد التعاقد بين الـ Repository وباقي المشروع، وبيسهّل التبديل أو الاختبار.

---

## 🧱 الخطوة 1: إنشاء الـ Product Model مع Migration

```bash
php artisan make:model Product -m
```

شرح:  
`-m` تعني إنشاء ملف Migration تلقائي مع الموديل.

### ✏️ في ملف Migration:

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->decimal('price', 10, 2); // السعر بفاصلة عشرية
    $table->timestamps();
});
```

ثم نشغّل الأمر:

```bash
php artisan migrate
```

---

## 🧱 الخطوة 2: إنشاء الـ Interface

```bash
mkdir app/Repositories
touch app/Repositories/ProductRepositoryInterface.php
```

```php
<?php

namespace App\Repositories;

interface ProductRepositoryInterface
{
    public function getAll();
    public function getById($id);
    public function create(array $data);
    public function update($id, array $data);
    public function delete($id);
}
```

---

## 🧱 الخطوة 3: إنشاء الـ Repository

```bash
touch app/Repositories/ProductRepository.php
```

```php
<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository implements ProductRepositoryInterface
{
    public function getAll()
    {
        return Product::all();
    }

    public function getById($id)
    {
        return Product::findOrFail($id);
    }

    public function create(array $data)
    {
        return Product::create($data);
    }

    public function update($id, array $data)
    {
        $product = Product::findOrFail($id);
        $product->update($data);
        return $product;
    }

    public function delete($id)
    {
        return Product::destroy($id);
    }
}
```

---

## ⚙️ الخطوة 4: ربط الـ Interface بالـ Repository في `AppServiceProvider`

افتح `app/Providers/AppServiceProvider.php` وأضف:

```php
use App\Repositories\ProductRepositoryInterface;
use App\Repositories\ProductRepository;

public function register()
{
    $this->app->bind(ProductRepositoryInterface::class, ProductRepository::class);
}
```

---

## 🧠 الخطوة 5: إنشاء الـ Service

```bash
mkdir app/Services
touch app/Services/ProductService.php
```

```php
<?php

namespace App\Services;

use App\Repositories\ProductRepositoryInterface;

class ProductService
{
    protected $productRepository;

    public function __construct(ProductRepositoryInterface $productRepository)
    {
        $this->productRepository = $productRepository;
    }

    public function getAll()
    {
        return $this->productRepository->getAll();
    }

    public function getById($id)
    {
        return $this->productRepository->getById($id);
    }

    public function create(array $data)
    {
        return $this->productRepository->create($data);
    }

    public function update($id, array $data)
    {
        return $this->productRepository->update($id, $data);
    }

    public function delete($id)
    {
        return $this->productRepository->delete($id);
    }
}
```

---

## 🎮 الخطوة 6: إنشاء الـ Controller

```bash
php artisan make:controller ProductController
```

```php
<?php

namespace App\Http\Controllers;

use App\Services\ProductService;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function index()
    {
        return response()->json($this->productService->getAll());
    }

    public function store(Request $request)
    {
        return response()->json($this->productService->create($request->all()));
    }

    public function show($id)
    {
        return response()->json($this->productService->getById($id));
    }

    public function update(Request $request, $id)
    {
        return response()->json($this->productService->update($id, $request->all()));
    }

    public function destroy($id)
    {
        $this->productService->delete($id);
        return response()->json(['message' => 'Deleted']);
    }
}
```

---

## 🛣️ الخطوة 7: إعداد الـ Routes

في `routes/api.php`:

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

---

## 📌 الترتيب النهائي:

1. **Interface:** تعريف قواعد التعامل.
2. **Repository:** تنفيذ العمليات مع قاعدة البيانات.
3. **Service:** يحتوي منطق العمل.
4. **Controller:** يستقبل الطلب ويرد بالنتيجة.
5. **Route:** يوصل بين الـ Endpoint و الـ Controller.

---

## 👨‍💻 Author

Muhammed Salama  
[devmuhammedsalama@gmail.com](mailto:devmuhammedsalama@gmail.com)