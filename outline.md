**第六节 Discovering and Configuring the Laravel Structure for the RESTful API**

_25 介绍Laravel 的目录结构_

_26 介绍Laravel 的PHP Artisan 命令行_

_27 介绍Laravel 的环境变量_

_28 介绍Laravel 路由系统_


**第七节 Creating the Initial Laravel Components for the RESTful API**

_29 遇到问题时如何处理_

_30 创建模型_

Buyer 和 Seller

    ```
    php artisan make:model Buyer
    php artisan make:model Seller
    ```
Buyer 和 Seller 扩展 User 模型
```
class Seller extends User <-- Model 改为 User
use Illuminate\Database\Eloquent\Model; <-- 删除这行
```

Category, Product, Transaction, 创建模型的同时创建 migration

    ```
    php artisan make:model Category -m
    php artisan make:model Product -m
    php artisan make:model Transaction -m
    ```

_31 创建控制器_

以 Resource 的方式创建控制器： -r
```
php artisan make:controller Buyer/BuyerController -r
php artisan make:controller Category/CategoryController -r
php artisan make:controller Product/ProductController -r
php artisan make:controller Seller/SellerController -r
php artisan make:controller Transaction/TransactionController -r
php artisan make:controller User/UserController -r
```

_32 用 Laravel Resource Route 创建访问路径_
修改了 route\api.php 文件

**第八节 Creating the Initial Laravel Components for the RESTful API**

_33 定义 Category 模型的属性_
修改了 app\Category.php 文件

_34 定义 Product 模型的属性_
修改了 app\Product.php 文件

_35 定义 Transaction 模型的属性_
修改了 app\Transaction.php 文件

97:缺少数据库
```
php artisan make:controller Seller/SellerTransactionController -r -m Seller
```
  
98:OK
```
php artisan make:controller Seller/SellerCategoryController -r -m Seller
```

99:缺少数据库
```
php artisan make:controller Seller/SellerBuyerController -r -m Seller
```

100:OK
```
php artisan make:controller Seller/SellerProductController -r -m Seller
```
保留除 crate, show, edit 以外的所有方法
显示Seller的所有Product

101:OK
SellerProductController 的store方法
依据seller的id保存/上传一个产品(保存图像部份还未完成)

102:
SellerProductController 的update方法

103:
SellerProductController 的delete方法

节25：
Product复杂的操作方法

104：缺少数据库
教程里并没有显示这个数据库里有数据
ProductTransaction 的 Index方法
```
php artisan make:controller Product/ProductTransactionController -r -m Product
```

105：缺少数据库
ProductBuyer 的 Index方法
```
php artisan make:controller Product/ProductBuyerController -r -m Product
```

106:OK
ProductCategory 的 Index方法

```
php artisan make:controller Product/ProductCategoryController -r -m Product
```

107:OK
ProductCategory 的 update方法

108:OK
ProductCategory 的 delete方法


节28: Middleware
127: OK 
Middleware　有global 和 group
在 Providers\RouteServiceProvider.php 里，可以定义使用哪个group的Middleware 
Middleware 可以接受参数
也可以 Controller　里使用 Middleware

128: OK 
创建 Middleware
```coffeescript
php artisan make:middleware SignatureMiddleware
```
完成后在kernel.php　登记这个 Middleware

129: OK
关于 rate limit

130: 5.5　不需要设置

节28: Transforming
131: OK
为什么要使用transform
spatie/laravel-fractal
132: OK
安装：
composer require spatie/laravel-fractal
安装后５.5会自动注册
运行 php artisan 会发现多了 make:transformer　指令
133: OK
创建 User transformer
```
php artisan make:transformer UserTransformer
```
134: OK
创建 Seller 和 Buyer transformer
```
php artisan make:transformer BuyerTransformer
php artisan make:transformer SellerTransformer
```

135: OK
创建 Category transformer
```
php artisan make:transformer CategoryTransformerP
```

136: OK
创建 transaction transformer
```
php artisan make:transformer TransactionTransformer
```

137: OK
创建 product transformer
```
php artisan make:transformer ProductTransformer
```

138: OK
使用transformer
在model里加入 public $transformer = UserTransformer::class;

139: OK
返回transform 过的数据
\app\traits\ApiResponser.php
```
    protected function showAll(Collection $collection, $code = 200)
    {
        if ($collection->isEmpty()) {
            return $this->successResponse(['data' => $collection], $code);
        }
        $transformer = $collection->first()->transformer;

        $collection = $this->transformData($collection, $transformer);
        return $this->successResponse($collection, $code);
    }
    
    protected function showOne(Model $instance, $code = 200)
    {
        $transformer = $instance->transformer;

        $instance = $this->transformData($instance, $transformer);
        return $this->successResponse($instance, $code);
    }

    protected  function transformData($data, $transformer)
    {
        $transformation = fractal($data, new $transformer);
        return $transformation->toArray();
    }
```

节30：排序和过滤
140: OK
\app\traits\ApiResponser.php
```
    protected function showAll(Collection $collection, $code = 200)
    {
        if ($collection->isEmpty()) {
            return $this->successResponse(['data' => $collection], $code);
        }
        $transformer = $collection->first()->transformer;

        $collection = $this->sortData($collection);
        $collection = $this->transformData($collection, $transformer);
        return $this->successResponse($collection, $code);
    }
    
    protected function sortData(Collection $collection)
    {
        if (request()->has('sort_by')) {
            $attribute = request()->sort_by;
            $collection = $collection->sortBy->{$attribute};
        }
        return $collection;
    }
```

141: OK
上面的排序因为经过了 transform 因此以identifier 作为排序参数时会得到预期之外的效果 

142：OK
transform后的字段和原始建一个map
在 transformer 文件：
    public static function  originalAttribute($index)
    {
        $attributes = [
            'identifier' => 'id',
            'name' => 'name',
            'email' => 'email',
            'isVerified' => 'verified',
            'isAdmin' => 'admin',
            'creationDate' => 'created_at',
            'lastChange' => 'updated_at',
            'deleteDate' => 'deleted_at',
        ];

        return isset($attributes[$index]) ? $attributes[$index] : null;
    }

143: OK
\app\traits\ApiResponser.php
```
    protected function sortData(Collection $collection, $transformer)
    {
        if (request()->has('sort_by')) {
            $attribute = $transformer::orginalAttribute(request()->sort_by);
            $collection = $collection->sortBy->{$attribute};
        }
        return $collection;
    }
```
144: OK
Filter

节31：Pagination

145: OK
eloquent本身可以用paginate() 代替all();

146: OK

147: 自定义 page size

节32：Cache
147: Cache

148: 让 Cache 分辨参数
先将url参数排序，再组合成fullUrl
***不确定Cache是否起了作用***

节33：HATEOAS支持
150:OK
什么是HATEOAS

151: OK
给 Category transformer 增加 hateoas 

152: OK
给 Product transformer 增加 hateoas

153: OK
给 Transaction transformer 增加 hateoas 

154: OK
给 User transformer 增加 hateoas

155: OK
给 Buyer transformer 增加 hateoas

156: OK
给 Seller transformer 增加 hateoas

节34：Transformation 和 验证
157：OK
当前，在 post 方法的验证时，错误提示的字段名是未经过 transform 的，这和显示的不一致

158：OK
创建一个 middleware 解决上一节的问题
```
php artisan make:middleware TransformInput
```
在kernel.php注册这个middleware

然后在Controller的construct
```coffeescript
    public function __construct()
    {
        parent::__construct();
        $this->middleware('transform.input:' . UserTransformer::class->only(['store', 'update']));
    }
```

节35: 用户验证
162：
web 和 api 两种不同的验证方式


节39：CORS
203:
关于 Cors
https://www.test-cors.org/

204: OK
安装 barryvdh/laravel-cors
```coffeescript
composer require barryvdh/laravel-cors
```

205: OK
设置 barryvdh/laravel-cors

config/app.php:
    /*
     * Package Service Providers...
     */
    \Barryvdh\Cors\ServiceProvider::class,

```
php artisan vendor:publish --provider="Barryvdh\Cors\ServiceProvider"
```
现在在config目录下有了cors.php可以自定义配置了。

206: OK
让Cors只应用于Rest api
app\http\
    protected $middlewareGroups
    'api' => [
        添加这行:
        'cors'
    ]
    
    protected $routeMiddleware = [
            添加这行:
            'cors' => \Barryvdh\Cors\HandleCors::class,
    ]
使用test-cors.org测试，(不要https)

207:OK
 
当异常发生时，Access-Control-Allow-Origin没有备添加到 response的头部，
这是因为　Cors middleware 还没有起作用。

比如: 当访问一个请求不存在的方法时：
api/buyer/1 的delete方法(buyer 的 delete 应该在 user 下完成)
’error‘: "The specified method for the request is invalid",
Access-Control-Allow-Origin没有加到Response　的　Headers

解决方案：
    app\Exceptions\Handler.php
    新增一个方法：
        public function handleException($request, Exception $exception)
    将原public function render($request, Exception $exception)的方法剪切复制到新方法中，
    在原方法中：
        $response = $this->handleException($request, $exception);
        app(CorsService::class)->addActualRequestHeaders($response, $request);
        return $response;