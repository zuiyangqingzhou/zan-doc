# ExceptionHandler

### 文件位置

```
resource/middleware/exceptionHandler.php
```

## 功能

自定义请求异常处理函数，当请求处理过程中抛出异常时，异常可以经过中间件处理后透传给调用方。

## 配置示例

### tcp

```php
<?php
use Zan\Framework\Network\Tcp\Exception\Handler\GenericExceptionHandler;

return [
    'match' => [
        [
            "/com/youzan/nova/framework/generic/service/GenericService/invoke", "genericExceptionHandlerGroup",
        ],
        [
            "/Com/Youzan/Nova/Framework/Generic/Php/Service/GenericTestService/ThrowException", "genericExceptionHandlerGroup",
        ],
        [
            ".*", "all"
        ]
    ],
    'group' => [
        "genericExceptionHandlerGroup" => [
            GenericExceptionHandler::class
        ],
        "all" => [
            GenericExceptionHandler::class
        ],
    ],
];
```

* match用于服务分组，服务名称支持正则表达式匹配。
* group设置分组名和对应的异常处理器，异常处理器需要实现接口Zan\Framework\Contract\Network\RequestFilter，一个分组名可以设置多个处理器。

 异常处理器的实现示例为：

```php
<?php
use Zan\Framework\Contract\Foundation\ExceptionHandler;

class GenericExceptionHandler implements ExceptionHandler
{
    public function handle(\Exception $e)
    {
        sys_error("GenericExceptionHandler handle: ".$e->getMessage());
        throw new \Exception("网络错误", 0);
    }
}
```

handle是个简单的函数，不是生成器，handle对异常进行处理，加工处理后需要透传给调用方的异常再次抛出或返回即可。

### http

http配置内容与tcp类似，不同点在于tcp以服务名和方法名为key，http以url的path为key（path未指明默认为/index/index/index）来匹配分组，http示例为：

```php
<?php
return [
     'match' => [
         [
            "index/index/index",  "all",
         ],
         [
            ".*", "all"
         ]
     ],
     'group' => [
         "all" => [
             TestExceptionHandler::class,
         ],
     ],
];
```

 异常处理器的实现示例为：

```php
<?php
use Zan\Framework\Contract\Foundation\ExceptionHandler;

class TestExceptionHandler implements ExceptionHandler
{
    public function handle(\Exception $e)
    {
        // 针对异常自行判断决定是否
        return new Response("网络错误");
        // or 
        return null; // 不做处理
    }
}
```

同样地，handle是个简单的函数，不是生成器。与tcp不同的是，handle对异常进行处理，加工处理后需要透传给调用方的异常需要返回。

