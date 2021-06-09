# PHP语法函数
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

## getenv
gentenv(参数)函数 是一个用于获取环境变量的函数，根据提供不同的参数可以获取不同的环境变量。

## strcasecmp
strcasecmp() 函数比较两个字符串，相等返回0，大于返回正数，小于返回负数。
## strip_tags
strip_tags() 函数剥去字符串中的 HTML、XML 以及 PHP 的标签。
## str_replace

```php
<?php
echo str_replace("world","Peter","Hello world!");
?>
// 把world换成Peter
```
## strpos
查找 "php" 在字符串中第一次出现的位置：
```php
<?php
echo strpos("I love php, I love php too!","php");
?> 
// 输出结果为7
```
## preg_replace
```
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```

## unset
unset() 函数用于销毁给定的变量。