# PHP特性
- [PHP::](PHP::)
    - [一个类例子](#一个类例子)
        - [使用非静态方法要先创建实例](#使用非静态方法要先创建实例)
        - [使用静态方法无需创建实例，直接使用类名](#使用静态方法无需创建实例，直接使用类名)
    - [静态方法与非静态方法区别](#静态方法与非静态方法区别)
- [PHP引号解析问题](#PHP引号解析问题)
- [PHP引用&](#PHP引用&)
- [public&&private&&protected](#public&&private&&protected)

- [ini_get_all](#ini_get_all)
    - [disable_functions](#disable_functions)
# PHP::
### 一个类例子
```php
class aaa{
static function ar(){
}
function br(){}
}
```

### 使用非静态方法要先创建实例
```php
$obj = new aaa();
$obj -> br();
```

### 使用静态方法无需创建实例，直接使用类名
```php
aaa::ar();
```

### 静态方法与非静态方法区别
- 静态方法不需要new实例，直接object::fun1使用

# PHP引号解析问题
## 前言
在PHP语言中，单引号和双引号都可以表示一个字符串，但是对于双引号来说，可能会对引号内的内容进行二次解释，这就可能会出现安全问题。

```php
<?php
$a = 1;
$b = 2;
echo '$a$b';//输出结果为$a$b
echo "$a$b";//输出结果为12
?>
```

# PHP引用&
- PHP 的引用允许你用两个变量来指向同一个内容
```php
<?php
$a = 23;
$b = &$a;
$b = 42;
var_dump($a); // int(42)
var_dump($b); // int(42)
?>
```


# public&&private&&protected
- public 表示全局，类内部外部子类都可以访问；
- private表示私有的，只有本类内部可以使用；
- protected表示受保护的，只有本类或子类或父类中可以访问


# ini_get_all
获取所有已注册的配置选项

```php
<?php
var_dump(ini_get_all());
```