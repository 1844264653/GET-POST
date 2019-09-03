### GET和POST到底有什么区别？

​	这个问题看起来很初级，但是实际上却涉及到方方面面，这就是为啥面膜你是老特么喜欢这个问题了，能真的说明白的真的比较少。



​	HTTP最早用来做浏览器与服务器之间交互HTML和表单的通讯协议；后来又呗广泛的扩充到接口格式的定义上。所以在讨论GET和POST区别时候，需要确定下到底是浏览器使用的GET和POST还是HTTP作为接口传输协议的场景。



#### 浏览器的GET和POST

​	这里特指浏览器中的非AJAX的HTTP请求，即从HTML和浏览器诞生就一直使用的HTTP协议中的GET/POST。浏览器用GET请求来获取一个html页面/图片/css/js等资源；用POST来提交一个<form>表单，并得到一个结果的网页。

​	浏览器将GET和POST请求定义为：

##### GET

​	‘读取’一个资源。比如Get到一个html文件。反复读取不应该对访问的数据有副作用。比如‘GET’一下，用户就下单了，返回订单已经受理，这是不可接受的【这谁顶得住啊】

​	**没有副作用被称为‘’幂等‘（idempotent）**

​	因为GET是读取，就可以对get请求的数据做**缓存**，这个缓存可以做到浏览器本身上——**彻底避免浏览器发请求**，也可以做到代理上——**nginx**，或者做到sever端——**Etag，至少可以减少带宽消耗**

~~~Etag
Etag 是URL的Entity Tag，用于标示URL对象是否改变，区分不同语言和Session等等。具体内部含义是使服务器控制的，就像Cookie那样。

HTTP协议规格说明定义ETag为“被请求变量的实体值”。另一种说法是，ETag是一个可以与Web资源关联的记号（token）。典型的Web资源可以一个Web页，但也可能是JSON或XML文档。服务器单独负责判断记号是什么及其含义，并在HTTP响应头中将其传送到客户端，以下是服务器端返回的格式：ETag:"50b1c1d4f775c61:df3"客户端的查询更新格式是这样的：If-None-Match : W / "50b1c1d4f775c61:df3"如果ETag没改变，则返回状态304然后不返回，这也和Last-Modified一样。测试Etag主要在断点下载时比较有用
~~~





##### POST

​	在页面里<form>标签会定义一个表单。点击其中的submit元素会发出一个POST请求让服务器做点事。这件事往往是有**副作用**的。

​	**有副作用是不’幂等‘**

​	**不幂等也就意味着不能随意多次执行。因此也就不能缓存**。因为post可能有副作用，所以浏览器实现为不能把POST请求保存为标签——点一下标签就下一个单，这谁顶得住啊？

​	此外，如果尝试重新执行POST请求，浏览器也会弹一个框提示这个刷新可能会有副作用，询问要不要继续。



​	当然了，服务器的开发者完全可以把GET实现为有副作用；把POST实现为没有副作用。只不过和浏览器的预期不符。**把GET请求实现为有副作用是个很可怕的事情**。之前百度贴吧有一个因为使用get请求可以修改管理员权限而造成的安全漏洞。反过来，把没有副作用的请求用POST实现，浏览器还是会弹框，对用户体验好处改善不大。

​	

​	但是后边可以看到，**将HTTP POST作为接口形式使用时，就没有这种弹框了。于是把一个POST请求实现为幂等就有实际意义了**。**POST幂等能让很多业务的前后端交互更加顺畅，以及避免一些因为前端的BUG，触控失误等带来的重复提交**。将一个有副作用的操作实现为幂等**必须重业务上能定义初怎么就算是’重复‘**。如果提交数据中增加一个dedupkey在一个交易会话中有效，或者利用提交的数据里可以天然当dedupkey的字段。这样万一用户强行重复提交，服务器端也可以做一次防护。

​	**ps：dedupkey：使用一个由客户端生成的，可用于避免重复的key**，俗称**dedup key**（deduplicate key之意），这个key可以用任意可以保证全局唯一性的方式生成，比如uuid。客户端和服务器需要使用这个dedup key作为串联条件，一起解决去重问题。

​	

​	GET和POST携带数据的格式也有区别。当浏览器发出一个GET请求时，就意味着要么时用户自己在浏览器地址栏输入，要么就是点击了html里的a标签的href中的url。所以其实并不是GET只能用url，而是浏览器直接发出的GET只能由一个url触发。所以没得办法，get上要在url之外带一些参数就只能依靠url上附带querystring。但是HTTP协议本身并没有这个限制。



​	浏览器的POST请求都来自表单提交。每次提交，表单的数据被浏览器用编码到HTTP请求的body里。浏览器发出的POST请求的body主要有两种格式，**一种是application/x-www-form-urlencoded**用来传输简单的数据，大概就是’key1=value1&key2=value2‘这样的格式。**另外一种是传文件，会采用multipart/form-data格式**。**采用后者是因为application/x-www-form-urlencoded的编码方式对于文件这种二进制的数据非常低效**



​	浏览器在POST一个表单时，url上也可以携带参数，只要<form action='url'>里的url带querystring就行。只不过表单里的那些用<input>等标签经过用户操作产生的数据都会在body里



​	因此我们一般会**泛泛的说’GET请求没有body只有url，请求数据放在querystring中；POST请求的数据在body中’**，但是！！！**这种情况仅仅限于浏览器发请求的场景！**



### 接口中的GET和POST

​	 这里是指的是通过浏览器的Ajax api，或者IOS/Android的App的http client， java的commands-httpclient/okhttp或者是curl， postman这几类的工具发出来的GET和POST请求。此实GET/POST不光能用在前端和后端的交互中，还能用在后端的各个自服务的调用中（即当一种RPC【远程过程调用】协议使用），尽管RPC有很多协议，比如thrift，grpc，但是http本身已经有大量的现成的支持工具可以使用，并且都对人类很友好，很容易debug。HTTP协议在微服务的使用是非常普遍的。



​	当用HTTP实现接口发送请求时，就没有浏览器中那么多限制了，只要是符合HTTP个格式的就可以发。HTTP请求的格式，大概是这样的一个字符串【\r\n换行了】：

~~~html
<METHOD><URL> HTTP/1.1\r\n
<Header1>:<HeaderValue1>\r\n
<Header2>:<HeaderValue2>\r\n
    ...
<HeaderN>:<HeaderValueN>\r\n
    \r\n
<Body Data...>
~~~

​	其中的<METHOD>keyishi GET也可以是POST，或者是其他的HTTP Method。比如PUT,DELETE,OPTION...。从协议本身看，并没有什么限制说GET一定不能没有body，POST就一定不能把参数放到<URL>的querystring上。因此其实可以更加自由的去利用个格式。比如Elastic Search的_search api就是用了带body的GET；也可以自己开发接口让POST一半的参数放在url的querystring里，另一半放在body里；你甚至还可以让所有的参数都放在Header里——可以做各种各样的限制，只要请求的客户端和服务器能够约定好。

~~~ps
 注：ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口
~~~



​	**当然太自由也带来了另一种麻烦**，开发人员不得不每次讨论确定参数是放url的path里，querystring里，body里，header里这种问题，太低效了。于是就有了一系列接口规范/风格。其中名气最大的当数REST。REST充分运用GET,POST,PUT和DELETE，约定了这4个接口分别获取，创建，替换和删除资源，**REST最佳实践还推荐在请求体使用json格式。**这样仅仅通过http的method就可以明白接口是什么意思，并且解析格式也得到了统一。

~~~markdown
json相对于x-www-form-urlencode的优势在于：
	1. 可以有嵌套结构；
	2.可以支更丰富的数据类型，通过一些框架，json可以直接被服务器代码映射为业务实体。用起来十分方便。但是如果是写一个接口支持上传文件，那么还是multipart/form-data格式更合适
~~~

​	REST中的GET和POST不是随便用的。在REST中，**GET + 资源定位符 被专用于获取资源或者资源列表**

比如：

~~~html
GET http://foo.com/books  --获取书籍列表--
GET http://foo.com/books/:bookId  --根据bookId获取一本具体的书--
~~~

​	与浏览器的场景相似，REST GET也不应有副作用，于是可以被反复无脑调用。浏览器（包括浏览器的Ajax请求）对于这种GET也可以实现缓存（如果服务器端提示了明确要Caching）；**但是如果用非浏览器**，有没有缓存完全看客户端的实现了。当然也可以从整个App角度，也可以完全绕开浏览器的缓存机制，实现一套业务定制的缓存框架（**可学习okhttp中控制Cache的类**）

​	

​	REST中的POST则用于创建资源  【POST】 + **资源定位符**

比如：

~~~html
POST http://foo.com/books
{
	"title:"you see you one day day",
	"author":"wsh",
	.....
}
~~~

​	这里就能留意到**浏览器中用来实现表单提交的POST，和REST里实现创建资源的POST语义上的不同**

~~~markdown
提下REST POST和REST PUT的区别。有些api是使用PUT作为创建资源的Method。PUT和POST的去别人在于，PUT的实际语义是“replace”replace。REST规范里提到的PUT请求应该是完整的资源，包括id在内。比如上面的创建一本书的api也可以定义为：
PUT http://foo.com/books
{
	"id":"BOOK:AFFE001BB0556A",
	"title:"you see you one day day",
	"author":"wsh",
	.....
}
服务器应该根据请求提供的id进行查找，如果存在一个对应的id的元素，就用请求中的数据*整体替换*已经存在的资源；如果没有，就用”把这个id对应的资源从【空】替换为请求数据“。——直观的看起来就是”创建“了。

	与PUT相比，POST更像一个”factory“，通过一组必要的数据创建出完整的资源。
	至于到底是用PUT还是POST创建资源，完全要看是不是提前可以知道资源所有的数据（尤其是id），以及是不是完整替换。
		比如对于AWS S3这样的对象存储服务，当想上传一个新资源时，其id就是”ObjectName“可以提前知道。这个api也总是完整的replace整个资源。这是的api用PUT的语义更合适；而对于那些id时服务器端自动生成的场景，POST更合适一些。
		
跑题了 打住
~~~



回到接口这个主题，上面仅仅粗略的介绍了REST的情况，但是现实中总是有REST的变体，也可能使用非REST的协议：比如JSON-RPC,SOAP等，每种情况的GET和POST又会有所不同。





### 关于安全性

​	我们常常听到GET不如POST安全，因为POST用body传输数据，而GET用url传输，更加容易看到。但是，**从攻击角度**，无论时GET还是POST都不够安全，因为HTTP本身就是**明文协议**。每个HTTP请求和返回的每个byte都会在网络上传播，不管时url，header还是body。**这完全不是一个‘是否在浏览器地址栏上看到’的问题**



​	为了避免传输中数据被窃取，**必须做从客户端到服务端的端端加密。业界通行的做法就时HTTPS——即用SSL协议协商出的密钥加密明文的http数据。**这个加密的协议和HTTP协议本身相互独立。如果时利用HTTP开发公网的站点/APP，要保证安全，https是最基本的要求。

~~~https
当然，端端加密并不一定非得用https。比如国内金融领域都会用私有网络，也有GB得加密协议SM系列。但是除了军队，金融等特殊机构之外，似乎没有必要自己发明一套类似于SSL的协议
~~~



​	回到HTTP本身，的确GET请求的参数更倾向于放在url上，因此会有更多的机会被泄露。比如携带私密信息的url会展示在地址栏上，还可以分享给第三方，就非常不安全了。此外，**从客户端都服务器端，有大量的中间节点，包括网关，代理等**。他们的access log通常会输出完整的url，比如**nginx的默认access log就是如此**。如果url携带敏感数据，就会被记录下来。**但是请注意，就算私密数据在body里，也是可以被记录下来的，因此如果请求要经过不信任的公网，避免泄密的唯一手段就是https**。这里说的“避免access log泄露”仅仅是***指避免可信区域中的http代理的默认行为带来的安全隐患。**比如你是不太希望让自己公司的运维同学从公司主网关的log里看到用户的密码吧

![](.\img\HTTPS.png)



​	另外，上面也讲过，如果是作为接口，GET实际上也可以带body,POST也可以在url上携带数据。所以实际上到底怎么传输私密数据，要看具体场景具体分析。当然，绝大多数场景，POST+body里写私密数据是合理的选择。一个典型的例子就是“登录”。

​	安全是一个巨大的主题，又有很多的细节组成一个完备体系，比如返回私密数据的mask，XSS，CSFR，跨域安全，前端加密，钓鱼，salt，... POST和GET在这件事上只是一个小角色，因此单独讨论POST和GET本身哪个更安全意义并不是太大。只要记住一般情况下，私密数据传输用POST+body就好。



### 关于编码