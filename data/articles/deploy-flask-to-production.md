
![图文无关，来源：Blue Archive](./images/Azusa20240902.png)

(↑ 图文无关，图片来源：Blue Archive / 白洲梓)

## 前言

Flask 应该还算是一个比较常用的 Web 框架，我此前也是用 Flask 开发过几个小项目，不过，怎么将 Flask 部署到生产环境中呢？

如果对于 Web 服务器的部署流程稍有了解的话，就应该知道我们肯定不能直接在目标服务器上执行一个 `pip install flask` 然后 `flask --app server run` 这样操作——毕竟我们开发的时候也是用的这条命令，然后在生产环境也是用的这条命令，考虑到生产环境和开发环境的差异还是比较大的，这不对吧（

的确不对，事实上，当我们在开发环境无数次地运行这条命令的时候，都会注意到上面有一行醒目的红字：

```
WARNING: This is a development server. Do not use it in a production deployment.
Use a production WSGI server instead
```

这其实已经给我们指明了方向，也就是说我们需要使用一个可用于生产环境的比较正式的 WSGI 服务器软件来在生产环境运行我们的 Flask 应用。如果要细究为什么一定要用 WSGI 服务器而不是直接用 Flask 的话，我总结了以下几点原因：

* 在默认的配置下，Flask 一次只能处理一个请求，所以显而易见地性能不好
* Flask 本身设计是在开发环境使用的，并不是为了在生产环境下稳定、安全、可靠的运行而设计的，如果要在生产环境使用，还是要用一个正规一点的 WSGI 服务器
* 而且，Flask 支持 Debugger 这些功能，通过 Debugger 可以实现任意命令执行，万一要是在生产环境不慎启用了，后果不堪设想！

## WSGI 服务器的选择和安装

WSGI，全称是 Python Web Server Gateway Interface，大体上就是将我们的 Python 环境和 Web 服务器进行通信的一种接口。我个人是把它理解为和 CGI 近似的东西，或者说是 CGI 的 Python 版本（我乱说的不一定对，只是我个人这样理解的）。不过这个其实不是重点，重点是我们要怎么去找到这个可以在生产环境使用的 WSGI 服务器软件。我们可以直接参考 [Flask 的官方文档](https://flask.palletsprojects.com/en/3.0.x/deploying/)：上面直接列出了几个比较流行的 WSGI 服务器软件，包括 Gunicorn 和 Waitress。

Gunicorn 官网：<https://gunicorn.org/>

Waitress 官网：<https://docs.pylonsproject.org/projects/waitress/en/stable/index.html>

那么 Gunicorn 和 Waitress 应该选哪个呢？我个人认为比较明显的是平台支持上的差异：

* Gunicorn 不支持 Windows，不过在 WSL 上也能运行
* 相比之下 Waitress 支持包括 Windows、Linux 在内的一些平台，所以兼容性更加广一点

除此之外，这两个我都使用过了，在我的场景下，我没有感受到明显的差异。我最终是选择了 Waitress，除了兼容性的因素之外，如果它名字叫 Maid 就更好了（不是）

安装和使用方法：

```bash
pip install waitress
waitress-serve --host 127.0.0.1 --port 8000 server:app
```

这里你可以注意到，我所监听的是 127.0.0.1，而不是 0.0.0.0 或者某个网站域名，这并不是意味着我学艺不精，不知道应该使用 0.0.0.0 或者指定一个域名才能监听来自公网的流量，而是因为我们接下来还需要设置一个至关重要的东西：反向代理。我们打算，通过反向代理监听某个域名，然后把请求转发给本地 127.0.0.1 的 Waitress 服务器。这并不是我故意想多找点事做，而是因为我们需要区分清楚应用服务器和 Web 服务器之间的差异。包括在 Waitress 的官方文档<ref url="https://docs.pylonsproject.org/projects/waitress/en/latest/reverse-proxy.html">Using Behind a Reverse Proxy - Waitress 官方文档</ref>中也有说：

> Often people will set up "pure Python" web servers behind reverse proxies, especially if they need TLS support (Waitress does not natively support TLS). Even if you don't need TLS support, it's not uncommon to see Waitress and other pure-Python web servers set up to only handle requests behind a reverse proxy; these proxies often have lots of useful deployment knobs.

翻译一下：

> 人们经常会在“纯 Python”网络服务器的后面设置一个反向代理，尤其是当他们需要 TLS 支持时（Waitress 本身不支持 TLS）。即使你不需要 TLS 支持，也常常会看到 Waitress 和其他纯 Python 网络服务器被设置为仅在反向代理后处理请求；这些代理通常有很多有用的部署选项。

简单来说，使用反向代理之后，允许我们在反向代理这一层进行很多更加高级和上层的配置，比如说这里提到的 SSL 支持，还有一些别的，比如 IP 过滤之类的。当然，这些功能不是说非要用反向代理否则就不能实现，比如说我们可以在 Flask 应用里面写一些全局的 IP 过滤器，可能也能实现过滤 IP 的效果。不过这个涉及到一个东西放在哪里更加合适的问题。通常来说，我们认为设置一个额外的反向代理服务器来实现这些功能，包括 SSL 支持等，是更加合适的。

## 选择和安装反向代理服务器

最常见的选项是 Nginx，当然还有些别的，比如 Apache 和 Caddy. 我是一个 Caddy 厨，因为 Caddy *非常* 方便。

* 使用 Caddy 启动的任何网站，它会自动帮你申请和续期 SSL 证书。不再需要配置 `acme.sh` 或者其它类似的任何东西，只要你的网站里指定了域名信息，它就会全自动地进行证书的获取和续期。
* Caddy 对于任何常见需求的写法都 *非常* 简洁。简单举个例子：如果我们需要启动一个静态文件服务器，需要几行代码呢？答案是只需要一行。

```
file_server
```

同样的，如果我们需要进行反向代理，需要几行代码呢，也是一行：

```
reverse_proxy localhost:8000
```

简而言之，使用 Caddy 可以让你省下配置 SSL 和研究配置文件写法的一大笔时间。当然这个东西也并非完美无缺，首先，复杂的配置肯定还是需要查文档的，当然文档本身读起来也不费劲，不过如果你要用 ChatGPT 之类的生成 Caddyfile 配置文件的话，你可能会失望了。这可能是因为 Caddy 属于较新的服务器软件，所以社区资料相比 Apache 和 Nginx 确实属于偏少的状态。当然你让 GPT 生成一段配置文件给你肯定能生成，但是能用吗，如能。大部分情况下你还是需要根据报错自己排查，查看文档，进行合适的修改的。因此，如果你比较依赖使用 GPT 进行配置生成和错误排查，可能 Nginx 和 Apache 会更适合你。

不管怎样，我这里是使用 Caddy 作为反向代理服务器。安装 Caddy 的方法不再说了，这里直接写怎么写配置文件。

默认的配置文件是在 `/etc/caddy/Caddyfile`，使用 `nano` 编辑它。

我们假设我们要部署到的域名是在 `example.com` 这里，并假设我们的 Waitress 是监听在 `127.0.0.1:8000`。那么我们的 Caddyfile 应该这么写：

```
example.com {
    reverse_proxy localhost:8000
}
```

使用`systemctl`重启一下`caddy`服务：

```
systemctl restart caddy
```

然后修改 DNS，你会发现 Caddy 自动给你的域名申请并使用了一个 SSL 证书。再次证明了使用 Caddy 可以为我们的服务部署提供很多便利。当然这不是重点。在现在这样的情况下，你的网站（在理论上）应该已经可以正常对外提供服务了。你可以试试访问网站的域名，看一看各种请求是不是都能够正常发送。

如果你像我一样使用 Cloudflare 对流量进行了代理，可能会发现自己的服务器怎么遇到了一些奇怪的问题（比如访问超时，或者无限 308 跳转之类），这时候，我们需要做一些小小的调整。

## 与 Cloudflare 相互配合

### SSL/TLS 模式的正确选择

首先，我们需要正视 Cloudflare 的 SSL/TLS 模式选择。这个东西绝对不是选择什么都无所谓，在不熟悉的情况下，我们对它的理解并不一定正确。比如，我一开始没有注意这个模式选择，直接采用了它的默认设置 `Flexible` (弹性, 灵活)，从而造成了一个困扰了我很久才修好的问题：无限 308 跳转。

这里贴出一些这种错误的症状：

* preflight is invalid (redirect)
* redirect is not allowed for preflight request
* (网站)将你重定向的次数过多，尝试清除 Cookies 后再试
* Too many redirects
* Error: Exceeded maxRedirects. Probably stuck in a redirect loop

TLDR：**如果你遇到了这种问题，那么很有可能是因为你的 Cloudflare 的 SSL/TLS 配置选择了默认的 Flexible，不妨试一试改成 Full 或 Full (Strict)，可能就好了。** 实际上，只要你的服务器正确配置了 SSL 证书，永远最正确的选择就是使用最严格的 Full (Strict) 模式。

实际上，这是一个 Cloudflare 的 SSL/TLS 模式非常容易遇到的一个问题（指无限308），并且 Cloudflare 也有一个专门的 TroubleShooting 页面来说明这个问题：<https://developers.cloudflare.com/ssl/troubleshooting/too-many-redirects/>

感谢 [scientificworld](https://koishi514.moe) 和 [Misaka13514](https://apeiria.net) 给我建议和帮我调试这个问题。

简单来说，我们可以查阅一下 Cloudflare 官方文档，探究一下 Flexible 和 Full / Full (Strict) 都有什么区别。

首先是 Flexible：

> ### Flexible encryption mode
> 
> If your domain’s encryption mode is set to Flexible, Cloudflare sends unencrypted requests to your origin server over HTTP.
> 
> Redirect loops will occur if your origin server automatically redirects all HTTP requests to HTTPS.

注意到 Flexible 模式下，Cloudflare 与源服务器通讯时使用的是 HTTP 流量而非 HTTPS.

然后是 Full 和 Full (Strict)：

> ### Full or Full (strict) encryption mode
> 
> If your domain’s encryption mode is set to Full or Full (strict), Cloudflare sends encrypted requests to your origin server over HTTPS.
> 
> Redirect loops will occur if your origin server automatically redirects all HTTPS requests to HTTP.

而相比之下 Full / Full (strict) 模式下，Cloudflare 与源服务器通讯时是使用的 HTTPS 协议进行通讯。

部分服务器软件环境下，比如在我的配置环境下，我的服务器会默认将所有的 HTTP 请求 308 到 HTTPS 上，如果我们没有套 Cloudflare CDN，那么客户端直接就是我们的电脑，这时候服务器返回的 308 理应是可以正常奏效的（也就是正确地让我们跳转到 HTTPS 而不是 HTTP），然而我们套了一层 Cloudflare CDN 之后我们客户端这里实际上始终都是 HTTPS 的，需要正确跳转到 HTTPS 的 Cloudflare 与我们服务器的通讯而不是我们与服务器的通讯。当然服务器发来的 308 请求并不会对此施加正确的影响，在这一点上， Cloudflare 的 Flexible 模式下始终是与我们的服务器使用 HTTP 通讯，不管服务器怎样要求 308 跳转，Cloudflare 与服务器这一段的通讯方式都不会改变（始终是 HTTP），也因此，相当于一直在与服务器进行 HTTP 的通讯，而服务器也一直指示希望采用 HTTPS 方式通讯，造成了无限的 308 跳转。解决方式也很简单，就是我们将 SSL/TLS 模式替换为 Full 或 Full (strict)，这样就强制 Cloudflare 在和我们的源服务器通讯的时候必须使用 HTTPS 协议。

### 获取客户端真实 IP

当我们使用反向代理的时候，由于中间多了一层跳转，所以假使我们 Flask 应用中写了类似 `request.remote_addr` 这样的代码，并不能正确获取到对端的 IP。对此，通常的解决办法是让反向代理服务器给应用程序通过其它方式（比如 HTTP 请求头）传输客户端的真实 IP。当然，这时候我们必须修改应用程序里面所有用到 `request.remote_addr` 的代码，这个是不可避免的。

这里我们采用的解决方案是让反向代理服务器（在我们的情况下是 Caddy）给 Waitress 设置一个额外的 HTTP Header：`X-Real-IP`，然后修改我们 Flask 应用中所有与 `request.remote_addr` 相关的代码，将相关逻辑修改为读取我们手动设置的 `X-Real-IP` 头。

首先是在 Caddy 这一侧，添加设置 `X-Real-IP` 头的相关逻辑：

```
reverse_proxy localhost:8000 {
    header_up X-Real-IP {remote_host}
}
```

然后修改我们的 Flask 应用的相关代码：（假设我们将对端 IP 存储在变量 `remote_ip` 中）

```
remote_ip = request.headers.get("X-Real-IP", request.remote_addr);
```

这行代码使用了 `request.headers.get`函数，第一个参数表示我们读取的 HTTP 头是哪个，第二个参数指示默认值（Default Value），也就是假使我们的应用程序没有获取到这个标头，那么仍然有一个默认值，而不是获取到 None 之类完全没有意义的值。

然后就好了？等等，我们外面还套了一层 Cloudflare 呢，我们这样获取到的虽然不是 127.0.0.1 这样的本地 IP，但是获取到的也是 Cloudflare 的 IP，仍然不是客户端的真实 IP。这会有什么问题呢？比如假如我们的应用逻辑中有针对每个 IP 设定的请求次数（QPS）限制之类的东西，我们本来是希望限制该用户（或者攻击者）不能够以超出我们限制的次数发送请求，但是现在这样限制的是 Cloudflare 的 IP 不能够超出我们限制地向我们发送请求，这和我们的本意完全不相符了。正常用户如果运气不好，中间经过的 Cloudflare IP 恰好被限制了，那即使他本人一个请求都没有发送，但就是没办法继续正常使用我们的服务。而攻击者如果想要重复请求我们的服务，根本不需要用换 IP 之类成本高昂的办法，毕竟中间经过的 Cloudflare IP 是什么都不一定呢，每次也可能会变动，相当于可以利用 Cloudflare 庞大的 IP 库绕过我们的 IP 限制。因此，我们必须想办法从 Cloudflare 的请求中获取到客户端原始 IP，然后传递给我们的应用服务。

经验丰富的朋友也可以看出来，在我们上面的网络结构（Waitress <- Caddy (Reverse Proxy) <- Cloudflare <- Client）中，变动代价最小的办法是修改 Caddy 的代码，使它向 Waitress 传递 X-Real-IP 的时候，传递客户端的真实 IP （而不是 Cloudflare 的 IP）。

要做到这点非常简单，根据 <https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip> 的文档，Cloudflare 会给每个它发出的请求附带一个 `CF-Connecting-IP` 的请求头，这里面是客户端的真实 IP。因此我们可以轻易地将上面的 Caddy 代码修改为如下的形式：

```
reverse_proxy localhost:8000 {
    header_up X-Real-IP {http.request.header.CF-Connecting-IP}
}
```

这样，代码应该可以在*正常*环境下正常工作了。不过有什么问题呢？考虑这样的攻击情况：假如攻击者通过某种方式知道或者猜到了我们的源站 IP，然后修改了自己的 `hosts` 文件，将我们绑定的域名与源站 IP 相关联，这样就可以直接通过源站 IP 来访问我们的服务了，我们的 Caddy 服务器还傻傻地以为它是从 Cloudflare 连入的，试图读取 `CF-Connecting-IP` 头，不过攻击者此时又不是通过 Cloudflare 连进来的，当然也不能保证 `CF-Connecting-IP` 头存在，甚至攻击者可以去故意篡改这个头，比如改成 `114.51.4.19` 这种恶臭 IP 来使我们的数据库变臭（恼），我们必须有办法抵抗这种攻击。

再次感谢 [Misaka13514](https://apeiria.net) 和 [scientificworld](https://koishi514.moe/) 在相关安全问题上对我的提醒，以及帮助我测试和调试。

要达成这点并不困难。Cloudflare 公布了自己 CDN 的 IP 段：

* 官网介绍页面：<https://www.cloudflare.com/zh-cn/ips/>
* IPv4：<https://www.cloudflare.com/ips-v4/>
* IPv6：<https://www.cloudflare.com/ips-v6/>

我们可以使用上述的列表创建一个白名单，然后修改 Caddyfile，让它验证请求是否来自 Cloudflare，假如不来自，则拒绝请求。示例代码：

```
example.com {
    @cloudflare_ips {
        remote_ip 173.245.48.0/20
        remote_ip 103.21.244.0/22
        ...(省略)
    }

    handle @cloudflare_ips {
        reverse_proxy localhost:8000 {
            header_up X-Real-IP {http.request.header.CF-Connecting-IP}
        }
    }

    handle {
        respond "Only allow connections from cloudflare." 403
    }
}
```

这样，在 Cloudflare CDN 关闭或者攻击者知道源站 IP 的情况下，访问我们的服务，只会提示 `Only allow connections from cloudflare.`，保证了我们的服务只能通过 Cloudflare 访问，也因此一定存在真实正确的 `CF-Coneecting-IP` 头。

到了这里配置差不多就完成了，不过我们可以再考虑一些特殊情况：Cloudflare Workers 和 WARP。假如攻击者通过 Cloudflare Workers 反代我们的网站，然后在 Worker 代码中尝试篡改 CF-Connecting-IP 头，会发生什么呢？考虑到 Cloudflare Worker 的 IP 段已经被我们刚刚拉入白名单了，攻击者是否会成功呢？

答案是否定的，事实上，Cloudflare 已经考虑过这种特殊情况，防止这种情况的发生。文档地址：<https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip-in-worker-subrequests>

> In same-zone Worker subrequests, the value of CF-Connecting-IP reflects the value of x-real-ip (the client’s IP). x-real-ip can be altered by the user in their Worker script.
> 
> In cross-zone subrequests from one Cloudflare zone to another Cloudflare zone, the CF-Connecting-IP value will be set to the Worker client IP address '2a06:98c0:3600::103' for security reasons.
>
> For Worker subrequests destined for a non-Cloudflare customer zone, the CF-Connecting-IP and x-real-ip headers will both reflect the client’s IP address, with only the x-real-ip header able to be altered.
>
> When no Worker subrequest is triggered, cf-connecting-ip reflects the client’s IP address and the x-real-ip header is stripped.

简单来说，攻击者的这种情况属于这里的第二段所描述的情况：攻击者试图从另外一个 Cloudflare Zone 连入。这时为了安全考虑，`CF-Connecting-IP` 的值会被设定为固定的 `2a06:98c0:3600::103`。这样的话如果对方使用 Cloudflare Workers 来反代我们的这个服务器，那么我们这边记录的 IP 都是那个固定的 IP，甚至可以用此来针对性屏蔽 Cloudflare Workers；即使不特意屏蔽，因为用 Cloudflare Workers 连进来的 IP 都是完全固定的，所以举个例子，假如我们为每个 IP 设置了请求次数限制（例如：每个 IP 每天最多登录失败 20 次）它仍然是生效的，而且相当于所有试图使用 Cloudflare Workers 的攻击者共享这 20 次限制，事实上并没有降低安全性。而对于 WARP 的情况，目前而言 WARP 的出口 IP 段是 104.28，因此目前还不涉及到这个问题。

## 结语

~~懒得写结尾了，以下总结由 GPT 生成：~~

在这篇文章里，我详细探讨了如何将 Flask 应用部署到生产环境，强调了使用合适的 WSGI 服务器的重要性。由于 Flask 自带的开发服务器并不适合生产使用，是推荐使用 Gunicorn 或 Waitress 作为生产环境的 WSGI 服务器。文章分析了这两者的优缺点，最终选择了 Waitress，并介绍了如何通过反向代理服务器（如 Caddy）来处理 SSL 证书和请求转发。

在与 Cloudflare 配合使用时，想要提醒读者选择适当的 SSL/TLS 模式，特别是避免使用默认的 Flexible 模式，以防止造成无限重定向的问题。此外，文章中还提供了获取客户端真实 IP 的方法，强调了在反向代理配置中确保请求来自 Cloudflare 的重要性，以提高安全性。

最后，文章提到通过白名单 Cloudflare 的 IP 段，能够有效防止直接访问源服务器，确保只有经过 Cloudflare 的请求被允许。这些措施共同提升了应用的安全性和稳定性，使得在生产环境中运行 Flask 应用更加可靠。