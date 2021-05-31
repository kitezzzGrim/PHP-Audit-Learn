# PHP语法函数
- [PHP引用&](#PHP引用&)

## PHP引用&
- PHP 的引用允许你用两个变量来指向同一个内容
```
<?php
$a = 23;
$b = &$a;
$b = 42;
var_dump($a); // int(42)
var_dump($b); // int(42)
?>
```

## array_map
- array_map() 函数将用户自定义函数作用到数组中的每个值上，并返回用户自定义函数作用后的带有新的值的数组。
```php
<?php
function myfunction($num)
{
   return($num*$num);
}

$a=array(1,2,3,4,5);
print_r(array_map("myfunction",$a));
?>
//输出：Array ( [0] => 1 [1] => 4 [2] => 9 [3] => 16 [4] => 25 )
```
## array_merge
array_merge() 函数用于把一个或多个数组合并为一个数组
## strip_tags
strip_tags() 函数剥去字符串中的 HTML、XML 以及 PHP 的标签。
## str_replace

```php
<?php
echo str_replace("world","Peter","Hello world!");
?>
// 把world换成Peter
```

## preg_replace
```
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```