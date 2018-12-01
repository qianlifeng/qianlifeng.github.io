---
title: RESTful api跨域认证
date: 2013-11-13 16:24:17
tags: [web]
---

本文主要介绍了 RESTful api 跨域认证的一些知识。包括了什么是跨域，跨域的危害，如何进行合理的跨域认证等。另文中如有不对的地方，欢迎指正.

# 前言

在正式介绍 Web Api 跨域认证之前，我想先举一个 web api 跨域认证的例子，就算是需求吧，也有利于后面讲解的时候有例子可说。

假设现在有两台服务器：<i class="fa fa-windows"></i> Server A 和 <i class="fa fa-windows"></i> Server B。 <i class="fa fa-windows"></i> Server A(域名：https://www.serverA.com/api/) 上面运行着一个提供 Web Api 的程序，此程序中的 api 都需要用户登录以后才能操作。<i class="fa fa-windows"></i> Server B(域名：http://www.serverB.com) 也有一个程序，专门用来消费这些 Web PI 。

ServerB 上面有一个登录操作，此操作会去请求 ServerA 上面的某个用户认证 api。要求:

    1. 成功请求此api
    2. 持久化用户的此次认证，以便用户的后续请求。

# 什么是跨域

对于上面的要求，如果你直接通过 ajax 请求此认证 api 显然是不行的。为什么？因为这个请求跨域了。那么什么请求才是跨域请求？

    所谓跨域请求是指请求一个与当前url协议不同,或者域名不同,或者端口不同的链接资源。

下面这个表格可以帮助理解什么样的请求是跨域请求。

<table class="table table-bordered table-striped"><tr><th>URL</th><th>是否跨域</th><th>说明</th></tr> <tr><td> http://www.scottqian.com/folder1/a.html<br/>http://www.scottqian.com/folder2/b.html </td><td>否</td><td>协议，域名，端口都相同</td></tr> <tr><td> http://www.scottqian.com:8000/folder1/a.html<br/>http://www.scottqian.com/folder2/b.html </td><td>是</td><td>端口不同</td></tr> <tr><td> https://www.scottqian.com/folder1/a.html<br/>http://www.scottqian.com/folder2/b.html </td><td>是</td><td>协议不同</td></tr> <tr><td> http://www.domain.com/folder1/a.html<br/>http://www.scottqian.com/folder2/b.html </td><td>是</td><td>域名不同</td></tr> <tr><td> http://blog.scottqian.com/folder1/a.html<br/>http://www.scottqian.com/folder2/b.html </td><td>是</td><td>主域相同，但是子域不同</td></tr> </table>

这种不能跨域请求的限制又称为：“[Same-Origin Policy](http://en.wikipedia.org/wiki/Same-origin_policy)”（同源策略）。值得注意的是**这种安全限制是 Javascript 保证的**，也就是说以后如果出来个新的浏览器端语言不带这种限制，那么你就可以随便请求不同域的资源。还有一个例子可以佐证，你直接使用 wget 命令请求 RESTful api 认证的页面同样会有结果返回。因为此处的请求不是由 javascript 发出，已经没有了同源策略的限制了。

# 跨域的危害

为什么要采用这种同源策略的限制呢？我们来模拟一下攻击场景。比如说某网站 A 有个 api`http://localhost:5000/api/getuser`，此 API 用来获得当前用户的登陆信息 [ 假设此 API 不需要认证操作 ] 。碰巧的是用户同时在一个新的标签页打开了网站 B。网站 B 下面有这么一段代码：

```html
<!DOCTYPE html>
<html>
    <head>
        <script src="http://codeorigin.jquery.com/jquery-2.0.3.min.js" type="text/javascript"></script>
    </head>
    <body>
        <input type="button" id="btnGetInfo" value="Get"></a>


        <script type="text/javascript">
            $(function(){
                    $("#btnGetInfo").click(function(){
                        $.get("http://localhost:5000/api/getuser/1")
                            .success(function(d){
                                alert(d);
                            });
                    });
            });
        </script>
    </body>
</html>
```

如果没有同源策略的限制，那么用户打开网站 B 的同时，网站 B 就可以悄悄获得用户在网站 A 的信息了。如果这个请求换成某个删除 API，那就是大问题了。

# 如何进行跨域请求

[RESTful](http://www.ruanyifeng.com/blog/2011/09/restful.html) api 大行其道的今天，我们需要跨域请求的需求也越来越多。如何进行安全的跨域请求呢？这里介绍几种方法：

- JSONP

  JSONP 依赖于这样一个事实：带有 src 属性的标签具有跨域的能力。

比如大家经常看到的各种 CDN 加速就不存在跨域的问题。JSONP 是通过 JS 动态构造这么一个带有 src 属性的标签，比如说 script，src 的链接请求远程的跨域 URL。然后服务器端返回 JSON 格式的数据，而又那么巧 JS 对 JSON 格式原生就支持，所以返回的 JSON 数据拿过来就能用了。Jquery 已经对 JSONP 提供了封装，使用起来和一般的 ajax 请求没多大区别，具体用法大家去搜索一下就是，一大把。

JSON 有一个缺点就是不能进行 POST 操作，从上面 JSONP 的原理我们很容易得出这个结论。如果你想进行 POST，可以使用下面一种方法。

- [CORS](http://www.w3.org/TR/cors/)(Cross-Origin Resource Sharing)

CORS 和 JSONP 不一样，JSONP 有点用了 little trick 的意思，而 CORS 则显得更加正式一些。一旦服务器允许了 CORS，那么客户端代码并不需要特殊的处理。同样还支持 GET,POST 等等方法。

CORS 的流程是这样的：

1. 通过 ajax 请求跨域资源

2. 浏览器检测到跨域请求，此时浏览器会发送一个 OPTION 类型的请求到服务器。该请求是个试探性的请求，意在告诉服务器端有一个跨域请求正在请求，此时服务器端会得到这个请求的信息（请求方法，源请求地址等）。

3. 服务器判断是否允许该请求。如果允许，则服务器端在响应头中添加:`Access-Control-Allow-Origin`标志。

4. 客户端接受到服务器端返回，如果成功则正式发起这个跨域请求，此时请求会成功。否则，如果服务端返回不允许，那么客户端的代码也不会被执行。

整个过程就是浏览器与服务器端的一个协商过程，这个过程是透明的，不需要我们写额外的代码。唯一需要我们做的就是在服务器端设置返回`Access-Control-Allow-Origin`标志。
关于如何在不同的服务器上设置这个标志，有个专门的网站已经写了专门的教程了。[http://enable-cors.org](http://enable-cors.org)

- Flash

你也可以把跨域请求放在 Flash 里面，这样就不用受同源策略的限制了。此方法我没用过，只是提一下。

# 为什么传统认证方式不行

解决了跨域问题，RESTful api 跨域认证就完成了一半了。认证的方式有很多很多，我这里说的传统认证方式是指通过 Cookie 来协助认证的方式，比如 ASP.net 里面的 Form 认证。这类认证大多是通过在 Cookie 写入登陆信息，然后浏览器发送请求后服务端再去验证 Cookie 是否存在，从而达到认证用户的目的。但是我们现在涉及到一个跨域的问题，而`Cookie是不能跨站共享的`。即使 RESTful 那边设置了 cookie，也不会到当前请求的域下面。到了第二次请求的时候，还是得不到认证信息。

# 如何进行跨域认证

- [HTTP 基本认证](http://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)

> 在 HTTP 中，基本认证是一种用来允许 Web 浏览器或其他客户端程序在请求时提供用户名和口令形式的身份凭证的一种登录验证方式。
> 在发送之前是以用户名追加一个冒号然后串接上口令，并将得出的结果字符串再用 Base64 算法编码。例如，提供的用户名是 Aladdin、口令是 open sesame，则拼接后的结果就是 Aladdin:open sesame，然后再将其用 Base64 编码，得到 QWxhZGRpbjpvcGVuIHNlc2FtZQ==。最终将 Base64 编码的字符串发送出去，由接收者解码得到一个由冒号分隔的用户名和口令的字符串。

HTTP 的好处就是基本所有的浏览器都支持。缺点就是也太基本了，它假设的前提是客户端和服务端的通信是建立在可信的通信上面的。不然用户的登录信息很容易被获取。如果加上 SSL，那么或许是一种简单可靠的方式。

- OAUTH

> OAuth（开放授权）是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。
> OAuth 允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的 2 小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth 让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。

现在使用 OAUTH 认证的网站很多，很多都是使用了新浪微博，QQ 的 AUTH 服务认证。使用 OAUTH 之所以安全，是因为他把登录的风险转移到了认证服务上。当前网站拿到的是一串 token，服务端只认 token 进行授权就行了。  
如果你开发的应用只是在小范围内使用，比如公司内部，那么其实此应用并不适合 OAUTH。如果你不是用现有的认证服务商，就要自己搭建一个认证服务。略显重量级了。

- 自定义认证

还有一种方法是自己定义认证。既然 Cookie 行不通，那么还有什么东西客户端能够拿得到的。响应头信息！前面提到的`Access-Control-Allow-Origin`就是一种响应头。如果我们将用户认证成功后的标志放到响应头然后传到客户端，客户端在第二次请求的时候将这个标志再放到响应头中传到服务器，服务器验证通过后再返回带有这种响应头的数据便可。总体和 HTTP 基本认证有些类似。也有不一样。主要是加密过程不再使用简单的 BASE64。

我目前使用的便是这种方式，代码是 asp.net mvc。上点干活。

```c#
public class TokenInspector : DelegatingHandler
{
    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        const string TOKEN_NAME = "X-JWT-Token";
        if (request.RequestUri.AbsolutePath == "/user/validate") return base.SendAsync(request, cancellationToken);
        if (request.Method == HttpMethod.Options) return base.SendAsync(request, cancellationToken);

        if (request.Headers.Contains(TOKEN_NAME))
        {
            string encryptedToken = request.Headers.GetValues(TOKEN_NAME).First();
            try
            {
                AuthToken token = AuthToken.Decrypt(encryptedToken);
                //bool isValidUserId = IdentityStore.IsValidUserId(token.UserId);
                bool requestIPMatchesTokenIP = token.IP.Equals(request.GetClientIP());

                if (!requestIPMatchesTokenIP)
                {
                    HttpResponseMessage reply = request.CreateErrorResponse(HttpStatusCode.Unauthorized, "Invalid identity or client machine.");
                    return Task.FromResult(reply);
                }
            }
            catch (Exception ex)
            {
                HttpResponseMessage reply = request.CreateErrorResponse(HttpStatusCode.Unauthorized, "Invalid token.");
                return Task.FromResult(reply);
            }
        }
        else
        {
            HttpResponseMessage reply = request.CreateErrorResponse(HttpStatusCode.Unauthorized, "Request is missing authorization token.");
            return Task.FromResult(reply);
        }

        return base.SendAsync(request, cancellationToken);
    }

}

```

DelegatingHandler 这个类用来拦截 HTTP 消息的，有点类似于 HTTPModule 的作用，不过 HTTPModule 是 IIS 的东西。而这个 DelegatingHandler 是属于 asp.net mvc 里面的。可以看到我们在每个请求的时候，都会去检查一下请求头中是否存在特定的头。如果没有或者认证失败，就返回认证失败。  
在用户认证的时候，我们只需要在响应头中添加特定的头信息进去，然后让客户端每次请求的时候附上此头信息即可。（通过 jquery 的全部 ajax 设定可以很容易做到）

```c#
AuthToken token = new AuthToken(user.UserName, HttpContext.Current.Request.UserHostAddress,DateTime.Now);
HttpContext.Current.Response.AddHeader("X-JWT-Token", token.Encrypt());
// must set this, otherwise jquery can't access X-JWT-Token header
HttpContext.Current.Response.AddHeader("Access-Control-Expose-Headers", "X-JWT-Token");
```

使用自定义的方式，我们还要考虑这个头的时效性问题，如何防止中间人攻击等等问题。不过已经不属于这次的内容了，基本的跨域认证已经可以完成了。
