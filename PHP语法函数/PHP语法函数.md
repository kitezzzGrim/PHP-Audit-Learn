# PHP语法函数
- [addslashes](#addslashes)
- [array_key_exists](#array_key_exists)
- [array_map](#array_map)
- [array_merge](#array_merge)

- [file_get_contents](#file_get_contents)

- [getenv](#getenv)

- [htmlspecialchars](#htmlspecialchars)

- [json_encode](#json_encode)
- [json_decode](#json_decode)

- [strcasecmp](#strcasecmp)
- [strip_tags](#strip_tags)
- [str_replace](#str_replace)
- [stripcslashes](#stripcslashes)
- [strpos](#strpos)
- [strrpos](#strrpos)
- [strtolower](#strtolower)
- [substr](#substr)

- [preg_replace](#preg_replace)

- [reset](#reset)

- [unset](#unset)

## addslashes 
- addslashes() 函数返回在预定义字符之前添加反斜杠的字符串。
- 预定义字符:
```php
单引号（'）
双引号（"）
反斜杠（\）
NULL
```

## array_key_exists
- 检查键名 "Volvo" 是否存在于数组中：
```php
<?php
$a=array("Volvo"=>"XC90","BMW"=>"X5");
if (array_key_exists("Volvo",$a))
   {
   echo "Key exists!";
   }
else
   {
   echo "Key does not exist!";
   }
?>
// 输出结果：Key exists！
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

# file_get_contents


```php
<?php
echo file_get_contents("test.txt");
?>
// 输出结果: This is a test file with test text.
```
## getenv
gentenv(参数)函数 是一个用于获取环境变量的函数，根据提供不同的参数可以获取不同的环境变量。


## htmlspecialchars
htmlspecialchars() 函数把预定义的字符转换为 HTML 实体。

预定义的字符:
```
& （和号）成为 &
" （双引号）成为 "
' （单引号）成为 '
< （小于）成为 <
> （大于）成为 >
```

例子
```php
<?php
$str = "This is some <b>bold</b> text.";
echo htmlspecialchars($str);
?>
// 输出为：This is some <b>bold</b> text.
```
把 < 和 > 转换为实体常用于防止浏览器将其用作 HTML 元素。当用户有权在您的页面上显示输入时，对于防止代码运行非常有用。

## json_encode
对变量进行 JSON 编码

```php
<?php
   $arr = array('a' => 1, 'b' => 2, 'c' => 3, 'd' => 4, 'e' => 5);
   echo json_encode($arr);
?>
// 输出结果: {"a":1,"b":2,"c":3,"d":4,"e":5}
```

## json_decode
PHP json_decode() 函数用于对 JSON 格式的字符串进行解码，并转换为 PHP 变量。

```php
<?php
   $json = '{"a":1,"b":2,"c":3,"d":4,"e":5}';

   var_dump(json_decode($json));
   var_dump(json_decode($json, true));
?>
// 输出结果: 
// object(stdClass)#1 (5) {  //对象
//     ["a"] => int(1)
//     ["b"] => int(2)
//     ["c"] => int(3)
//     ["d"] => int(4)
//     ["e"] => int(5)
// }

// array(5) {                 //数组 true
//     ["a"] => int(1)
//     ["b"] => int(2)
//     ["c"] => int(3)
//     ["d"] => int(4)
//     ["e"] => int(5)
// }
```
## strcasecmp
strcasecmp() 函数比较两个字符串，相等返回0，大于返回正数，小于返回负数。
## strip_tags
strip_tags() 函数剥去字符串中的 HTML、XML 以及 PHP 的标签。

```php

<?php
echo strip_tags("Hello <b>world!</b>");
?>
// 输出结果: Hello world!
```
## str_replace

```php
<?php
echo str_replace("world","Peter","Hello world!");
?>
// 把world换成Peter
```

## stripcslashes
stripcslashes() 函数删除由 addcslashes() 函数添加的反斜杠。`\`
## strpos
查找 "php" 在字符串中`第一次`出现的位置：
```php
<?php
echo strpos("I love php, I love php too!","php");
?> 
// 输出结果为7
```

## strrpos
查找 "php" 在字符串中`最后一次`出现的位置：

```php
<?php
echo strrpos("I love php, I love php too!","php");
?> 
// 输出结果为19
```

## strtolower
把所有字符转换为小写
```php
<?php
echo strtolower("Hello WORLD.");
?>
// 输出结果为hello world.
```

## strtoupper
把所有字符转换为大写：

```php
<?php
echo strtoupper("Hello WORLD.");
?>
// 输出结果为HELLO WORLD.
```


## substr
substr() 函数返回字符串的一部分。
```php
//substr(string $string, int $start, int $length = ?): string

<?php
echo substr("Hello world",6);
?>
//输出world
```
## preg_replace
```
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```


## reset

reset() 函数将内部指针指向数组中的第一个元素，并输出。

```php
<?php
$people = array("Bill", "Steve", "Mark", "David");

echo current($people) . "<br>";
echo next($people) . "<br>";

echo reset($people);
?>

// 输出结果:
// Bill
// Steve
// Bill
```
## unset
unset() 函数用于销毁给定的变量。