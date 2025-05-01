# 🏁 أولاً: ليه نستخدم (Interface, Repository, Service) من الأساس؟

## ✅ الهدف:
نفصل المسؤوليات، بحيث:

- **Controller:** يستقبل الطلب ويتحكم في "السيناريو".
- **Service:** ينفذ المنطق الرئيسي (business logic).
- **Repository:** يتعامل مع قاعدة البيانات.
- **Interface:** يعرّف قواعد التعامل، بحيث تقدر تبدل أو تغيّر مصدر البيانات في أي وقت بسهولة.

## 🧱 الخطوة 1: إنشاء الـ Product Model
```bash
php artisan make:model Product -m
شرح:
make:model: أمر بيقول للـ Laravel: "اعمللي موديل".

-m: ينشئ ملف Migration معاً لبناء جدول المنتجات.

🧱 الخطوة 2: نفتح ملف الـ Migration ونعدّل جدول المنتجات
php

Schema::create('products', function (Blueprint $table) {
    $table->id(); 
    $table->string('name');
    $table->decimal('price', 10, 2); // السعر بفاصلة عشرية
    $table->timestamps(); 
});
ليه بنكتب كده؟
Laravel بيوفر طريقة سهلة لتعريف الجداول من غير ما تكتب SQL يدوياً.

بعد كده:

php artisan migrate
ينشئ الجداول في قاعدة البيانات.

🧱 الخطوة 3: نبدأ بالـ Interface
📂 أول حاجة نعمل فولدر اسمه Repositories في app

mkdir app/Repositories
touch app/Repositories/ProductRepositoryInterface.php
نكتب فيه:
<?php

namespace App\Repositories;

interface ProductRepositoryInterface
{
    public function getAll(); // هترجع كل المنتجات
    public function getById($id); // منتج واحد بالـ ID
    public function create(array $data); // إنشاء منتج
    public function update($id, array $data); // تعديل منتج
    public function delete($id); // حذف منتج
}
ليه عملنا كده؟
الـ Interface يحدد الوظائف اللازمة لأي repository، من غير تفاصيل تنفيذ.

🧱 الخطوة 4: نعمل الـ Repository الحقيقي

touch app/Repositories/ProductRepository.php
ونكتب:

<?php

namespace App\Repositories;

use App\Models\Product;

class ProductRepository implements ProductRepositoryInterface
{
    public function getAll()
    {
        return Product::all(); // يجيب كل المنتجات
    }

    public function getById($id)
    {
        return Product::findOrFail($id); // منتج واحد أو يرجّع 404
    }

    public function create(array $data)
    {
        return Product::create($data); // إنشاء جديد
    }

    public function update($id, array $data)
    {
        $product = Product::findOrFail($id);
        $product->update($data); // تعديل البيانات
        return $product;
    }

    public function delete($id)
    {
        return Product::destroy($id); // حذف المنتج
    }
}
شرح:
ننفذ الوظائف المعرفة في الـ Interface، بالتعامل مع Model.

⚙️ الخطوة 5: نربط الـ Interface بالـ Repository في AppServiceProvider
ليه؟
ليعرف Laravel كيف يحقن ProductRepositoryInterface.

افتح:


app/Providers/AppServiceProvider.php

use App\Repositories\ProductRepositoryInterface;
use App\Repositories\ProductRepository;

public function register()
{
    $this->app->bind(ProductRepositoryInterface::class, ProductRepository::class);
}
🧠 الخطوة 6: نعمل Service لمنتجاتنا

mkdir app/Services
touch app/Services/ProductService.php
واكتب فيه:

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
شرح:
الـ Service ينظم منطق التعامل مع الـ Repository بشكل منظم.

🎮 الخطوة 7: نعمل Controller

php artisan make:controller ProductController

واكتب:

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
        $products = $this->productService->getAll();
        return response()->json($products);
    }

    public function store(Request $request)
    {
        $product = $this->productService->create($request->all());
        return response()->json($product);
    }

    public function show($id)
    {
        $product = $this->productService->getById($id);
        return response()->json($product);
    }

    public function update(Request $request, $id)
    {
        $product = $this->productService->update($id, $request->all());
        return response()->json($product);
    }

    public function destroy($id)
    {
        $this->productService->delete($id);
        return response()->json(['message' => 'Deleted']);
    }
}

🛣️ الخطوة 8: نضيف الـ Routes

افتح routes/web.php أو routes/api.php لو هتشتغل API:

use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);

الترتيب:

Interface: تعريف التعامل مع البيانات.

Repository: الكود الفعلي للتعامل مع قاعدة البيانات.

Service: تنظيم المنطق الرئيسي للتعامل مع الـ Repository.

Controller: استقبال الطلبات وتوجيهها للـ Service.

👨‍💻 Author
Muhammed Salama
devmuhammedsalama@gmail.com
