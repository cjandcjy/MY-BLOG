- webpack熟悉吗? 抽取css的插件是什么？还有什么其他常用的插件？

    1. extract-text-webpack-plugin(https://webpack.docschina.org/plugins/extract-text-webpack-plugin)
    2. commonsChunkPlugin (https://webpack.docschina.org/guides/code-splitting/#%E9%98%B2%E6%AD%A2%E9%87%8D%E5%A4%8D-prevent-duplication-)
    3. HashedModuleIdsPlugin：使用comonsChunkPlugin将代码分割成应用代码和公共代码代码的时候，为了保持应用代码修改不影响公共代码的文件hash，需要用到这个插件（https://webpack.docschina.org/plugins/commons-chunk-plugin/）

- webpack 优化方法 ？ （https://zy108830.gitbooks.io/webpack-doc/content/optimization.html）

   1. 通过entry、output配置实现代码分离 ， 并用commonChunkPlugin防止代码重复
   2. 通过import()异步加载建立分割点，实现动态加载 ； 并且通过new webpack.optimize.LimitChunkCountPlugin({maxChunks: 15})和new webpack.optimize.MinChunkSizePlugin({minChunkSize: 10000})来防止因为分割点过多导致生成太多细小的模块
   3. 分割公共代码和应用代码
   4. 抽取css到单独的文件
   5. vue或者react都有自己的延迟加载机制，比如vue的路由懒加载


- SPA应用第一次加载获取打包文件太多如何优化？ 如何异步加载？(https://webpack.docschina.org/guides/code-splitting/#%E5%8A%A8%E6%80%81%E5%AF%BC%E5%85%A5-dynamic-imports-) （https://router.vuejs.org/zh/guide/advanced/lazy-loading.html）
- (https://webpack.docschina.org/guides/lazy-loading/#%E6%A1%86%E6%9E%B6)

    1. 可以使用webpack的code spliting功能， 将代码分割，并且搭配commonsChunkPlugin抽取公共代码， 这样可以分成几个打包文件 ， 减小bundle的大小。
    2. 比如Vue， 结合Vue的异步组件和webpack的code spliting完成路由懒加载


- 有读过vue或者react的源码吗？ 现在用的什么版本的Vue ， 知道Vue1和Vue2的区别吗？(https://cn.vuejs.org/v2/guide/migration.html)

    1. 
    vuejs1.0
    create -> beforeCompile -> compiled -> ready -> beforeDestroy -> destroyed

    vuejs2.0
    beforeCreate -> created -> beforeMount -> mounted -> beforeUpdate -> updated -> beforeDestroy -> destroyed

    1. 
    去除$dispatch与$broadcast
    使用emit与on的方法

    
    1. 绑定一次
    {{*msg}}
    <div v-once>{{msg}}</div>
    vue2.0已废弃 请使用v-once

    1. 绑定html代码
    {{{msg}}}
    <div v-html="msg"></div>
    {{{msg}}}写法vue2.0已废弃,请使用v-html

    1. 循环v-for
    数组
    1.0默认通过value进行遍历(key,value),遍历需加track-by="$index"(不加重复数据不绑定)

    2.0通过key进行遍历(value,key)

    ```
    data:{
    arr:['苹果','橘子','香蕉']
    }
    <ul>
    <li v-for="value in arr">
        {{value}} {{$index}}
    </li>
    </ul>
    ```
    {{{$index}}}写法vue2.0已废弃

- Vue组件间通信的方式 ？ 兄弟节点间怎么通信 ？

    1. provide/inject 可以让祖先组件向所有后代组件注入依赖，有点像react的上下文

    2. 通过 v-on 在子组件上绑定方法 ， 子组件中通过 this.$emit()去触发

    3. 事件广播的形式
  

- 数组扁平化能想到几种方法？ (https://github.com/mqyqingfeng/Blog/issues/36)
   
   递归 、toString 、 reduce 、 扩展运算符


- 数据库读写分离是什么？ 主从数据库? 分库分表呢？   （https://blog.csdn.net/justdb/article/details/17331569）（https://www.jianshu.com/p/000dfd9bc3cf）（https://www.jianshu.com/p/ab704b437ebd）
    
    读写分离的实现是靠主从服务器实现，主服务器不执行读操作只执行写操作，  从服务器只执行读操作（主服务器写操作之后，从服务器也会做同步， 同步方式有两种，一般是通过binarylog二进制log文件获取主库的操作进行同步）。
    实现读写分离的实现方式有两大类：
    1. 应用层：比如springjdbc、mybatis、hibernate通过配置多个数据源，一个用于写，其他用于读，这个方式在一主一从的时候很好实现，但是如果是一主多从，还需要通过一定的算法将读操作分摊到多个从库 ； 这样的优势是配置简单，避免了添加代理的复杂配置，并且少了代理层性能更好，不过将读写分离强耦合到业务代码中，不利于扩展
    2. 通过mysql-proxy这样的代理：在应用层和数据库之间新加一层代理， mysql请求都打到代理上，代理进行读写分配和负载均衡，这样的实现较为复杂，但是完全屏蔽了读写分离的实现，是比较好的工程实现方式
    3. 通过HAProxy：HAProxy工作在TCP/IP层，也需要应用层配合， 将读写分配给两个端口，一个端口写，一个端口读，读操作会由HAProxy进行负载均衡

- 数据一致性? （http://www.importnew.com/20633.html）（https://www.zhihu.com/question/31346392）
   
   首先分两种情况， 一种是数据库事务的一致性 ， 一种是分布式系统的数据一致性 
   1. 数据库事务具有ACID特性，原子性、一致性、隔离性、持久性， 隔离性是指不同事务之间相互影响的程度，可能造成：
    1. 脏读：一个事务读取了另一个事务修改了但是没有提交的更新，这个更新就是脏数据
    2. 幻读： 一个事务第一次读取了某结果， 第二次又读取发现多了数据，这是因为期间另一个事务提交了新的数据，这个叫幻读
    3. 不可重复读： 一个事务两次读到的数据不一样
    4. 第一类丢失更新： 一个事务的撤回覆盖了另一个事务的更新
    5. 第二类丢失更新： 一个事务的提交覆盖了另一个事务的更新
    其中一致性指的是数据库系统从一种一致的状态转换到另一种一致的状态 ， 事务可以有不同的一致性， 
    1. 强一致性： 读操作可以立即读到提交的更新
    2. 弱一致性： 读操作不一定可以立即读到提交的更新，有一个时间间隔， 叫不一致窗口
    3. 最终一致性： 事务更新一份数据，最终一致性保证在没有其他事务更新相同数据的情况下， 所有事务可以当读到之前事务的更新

  2. 分布式系统的一致性指的是分布式集群中各节点上的数据的一致和正确性


- 重放攻击 ， CSRF攻击说一下 ( https://blog.csdn.net/heluan123132/article/details/74375304)

    重放攻击是指攻击者发送一个目的主机已经收到过的包，来达到欺骗主机的目的，常用于身份验证 ； 攻击者通过网络监听或者其他方式盗取身份凭证，然后将其重新发送给认证服务器。
    防范措施：
    1. 请求中增加时间戳， 服务器只处理它认为的离当前时刻一定时间内的请求 ，这个可以防范时间戳过时的重放请求
    2. 通信双方商定一个序号，像TCP三次握手中的req-number和ack_number一样，每次请求这个数变化，以判断请求的新鲜性
    3. 提问和应答：A如果想接受B的信息 ， 提前给B一个N ， B提交数据的时候一起提交N或者f(N) ， A取出B对应的N或者f(N)和上传的数据进行对比， 一样就算验证通过

    针对上面的第三点的实践：
    进入登录页面的时候，服务器给客户端一个盐值，客户端输入的用户名密码和这个盐值一起md5运算后提交 ， 服务器查询用户名密码，并从客户端对应的session中获取盐值，进行md5运算后和参数进行比对。

    再进一步：
    用户名密码太简单的话，如果数据库数据泄露，黑客可以通过暴力方式获得密码原文， 但是又不想限制密码的复杂度， 所以可以在服务器中提供一个复杂的盐值，将用户密码和盐值拼接后的md5存到数据库中
  
   
 

- https 用过吗？知道什么原理吗？
  
  https超文本传输安全协议指的是工作一层加密连接（SSL或TSL）上的常规http连接的称呼 ， SSL\TSL叫做加密协议，位于http和tcp之间，它为http程序提供一个套接字供数据写入，加密之后进入传输层套接字 ， 或者从传输层获取加密数据，解密后交给http层。https默认使用443端口。在浏览器和服务器建立https连接的时候会发生四次握手：
  1. 浏览器首先向服务器发送 支持的SSL\TSL协议版本 、 一个随机数 、 可以接受的加密算法
  2. 服务器确认协议版本、加密算法， 生成随机数 ，向浏览器发送数字证书、随机数
  3. 浏览器生成第三个随机数 pre-master-key， 通过数字证书中公钥加密后发送给服务器，同时发送握手结束标志
  4. 服务器用私钥解密随机数 ， 发送握手结束标志，  之后双方通信就通过依靠三个随机数生成的会话秘钥进行加密

  非对称加密只在第三个随机数的时候使用，因为RSA这样的非对称算法比较耗时，而会话秘钥是对称加密。 但是因为整个握手过程除了第三个随机数机密，其他都是明文的，黑客可以获得前两个随机数以及加密算法， 如果破解了第三个随机数就没有 安全可言了，虽然理论上2048位的RSA算法是不能被破解的， 为了防止这种可能，在第三步中可以不发送第三个随机数，而是用DH算法，双方交换各自的参数算出这个随机数

  使用https的理由，是http有三个风险：
  1. 被窃听，因为全程明文， https通过会话秘钥加密可以避免
  2. 被篡改，报文在传输过程中被修改 ， https有校验机制可以避免
  3. 被伪造，第三方冒充他人身份进行通信， https配备身份证书可以避免


- post请求的content-type常用类型， 什么情况下选用什么类型

    1. applicant/x-www-form-urlencoded , form标签在不修改enctype的情况下默认是这个类型， 对键值对参数进行url编码
    2. applicant/json, restful接口常用的格式， 告诉服务器数据主体是序列化后的json数据
    3. text/xml  一般XML-RPC远程调用后端接口的时候使用
    4. multipart/form-data：  使用表单上传文件的时候的类型， 必须让 <form> 表单的 enctype 等于 multipart/form-data , 发送的时候会在content-type请求头中附上生成的一个boundary字符串， 用于分割表单中不同的字段， 每个字段前面都会由这个boundary分界，  最后以一个boundary结尾
  
 


- script标签的src加载 以及 text加载的时机？
    两者都是在挂载到dom树上之后才开始加载或者执行， src是异步的 ， text是同步的

    ```
    let script = document.createElement('script')
    script.src = 'http://xxxx.cdn.xxx_react.js'
    console.log('before append')
    document.body.appendChild(script)
    console.log('after append , ' , window.React) //输出 'after append , undefined'
    ```

    ```
    let script = document.createElement('script')
    script.text = 'console.log("this is script")'
    console.log('before append')
    document.body.appendChild(script)
    console.log('after append')   
    //输出：
    //before append
    //this is script
    //after append
    ```

- 没有挂载到dom树上的script或者link会发请求吗
  
  不会


- 说一下redis和mongodb的区别？ 如何选择？


- 有用过canvas吗？SVG、Canvas、D3.JS、WEBGL、Highcharts。
   
   1. canvas:(https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_usage)

- web动画实现的几种方式？

  1. transition
  2. @keyframes\animation
  3. canvas

- 说一下你项目中的登录验证过程？单点登录SSO的原理？(https://www.cnblogs.com/ywlaker/p/6113927.html)(https://github.com/hezhii/nodejs-sso-example/blob/56632d1062959ceede9eedb5435998536902b8ba/system/routes/index.js)

  SSO单点登录就是在多系统组成的应用群里， 实现一个子系统登录、所有子系统免登录 ， 一个子系统注销、所有子系统注销的效果， 实现依赖于一个SSO认证中心，用户的直接登录验证在这个中心完成，假设有两个子系统和一个单点登录认证中心：
  a.com , b.com , sso.com

  1. 用户请求 a.com/xxx , 检查session是否有用户信息，有则直接返回用户请求， 没有需要继续验证是否带有token，没有则跳转到登录页面 ：sso.com/login?redirectUrl='a.com/xxx',有则到sso中心去验证token，即请求sso.com/check_token?token='xxx' ， 检查这个请求的返回结果， 有用户信息就设置局部session，并返回用户请求 ， 没有则跳转到登录页面

  2. 用户到达登录页面输入用户名密码之后， 请求 sso.com/login?redirectUrl='a.com/xxx'的post接口，带上用户信息  认证中心验证通过，生成token，设置全局session，请求转发到redirecUrl指定的地址并带上token: a.com/xxx？token='yyyy' , a站收到请求后发现有token，请求sso.com/check_token?token='yyy'验证token的有效性， 验证成功的话则设置局部session，返回请求

  3. 用户请求 b.com/xxx， b站发现没有登录，重定向到 sso.com/login?redirectUrl=b.com/xxx， 因为a站登录的时候sso认证中心已经有了用户的全局session，取出token跳转到 b.com/xxx?token='yyy', b站收到token再去sso中心 sso.com/check_token?token='xxx'  验证token，验证成功的话设置局部会话， 返回结果
  
  （假设认证中心和系统2的url分别是：sso.com、system2.com，访问system2.com时因未登录而跳转到sso.com，跳转地址：http://sso.com?service=http://system2.com（不需要额外信息），此时，就变成了浏览器与http://sso.com站点之间的会话，这个会话因为系统1登录的原因已经被标记为已登录，所以认证中心取一块令牌，根据service参数回跳，并附上令牌，回跳地址：http://system2.com?token=token）


- 说一下你项目中的权限设计？
  

  

- 知道async ， await的原理吗？了解Generator吗 
  async\await的原理参见：(http://www.ruanyifeng.com/blog/2015/05/async.html)
  generator参见：（http://www.ruanyifeng.com/blog/2015/04/generator.html）（https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators）
  


- js实现链式函数 ， 写出a、b、c三个函数， 使得a().b().c()的输出结果是a、b、c



- truck函数? 写出符合下面输出的函数？ (http://www.cnblogs.com/coco1s/p/6509141.html)
  
    ```
    func(1)(2)(3).valueOf() //6
    func(3)(4)(3).valueOf() //10
    func(1)(2，5).valueOf() //8
    ```


- 什么是函数式编程？什么是不可变数据结构？ 有什么用该怎么用？
    
    https://www.zhihu.com/question/28016223

    https://github.com/xufei/blog/issues/19

    不可变数据结构的大致理念是，任何一种赋值，都应当被转化成复制，不存在指向同一个地方的引用。比如说：

    ```
    var a = 1;
    var b = a;
    b = 2;
    console.log(a==b);
    ```

    这个我们都知道，b跟a的内存地址是不一致的，简单类型的赋值会进行复制，所以a跟b不相等。但是：

    ```
    var a = {
        counter : 1
    };
    var b = a;
    b.counter++;
    console.log(a.counter==b.counter);
    ```
    这时候因为a和b指向相同的内存地址，所以只要修改了b的counter，a里面的counter也会跟着变。Immutable Data的理念是，我能不能在这种赋值情况下，直接把原来的a完全复制一份给b，然后以后大家各自变各自的，互相不影响。

  