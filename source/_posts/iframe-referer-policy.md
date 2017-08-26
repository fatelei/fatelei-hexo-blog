title: Iframe referer policy
date: 2017-08-25 10:56:18
tags: [security, iframe]
---

在 iframe 中可以通过 `referrerpolicy` 指定获取指定资源时设置什么样的 [referrer](https://en.wikipedia.org/wiki/HTTP_referer)。

<!-- more -->

在 iframe 中 `referrerpolicy` 的取值有：

- no-referrer
- origin
- origin-when-cross-origin
- unsafe-url
- no-referrer-when-downgrade

### no-referrer

表示在访问资源的时候不会带上 `referrer` 信息

![no referrer](/images/no_referrer.png)

### origin

表示访问资源的时候，会带上来源页面的信息，包括：host 和 port

![origin](/images/origin.png)

### origin-when-cross-origin

表示访问不同域的资源，referrer 信息只包含 host 和 port，访问同域的资源时，除了 host 和 port，还会带上源页的 path 信息

- 不同域

![not same origin](/images/cross_origin.png)

### unsafe-url

表示访问资源时，referrer 信息会带上除 fragment、password、username 以外的所有信息，包括：host、port、path，当使用这个 policy 会有安全隐患，会把受 TLS 保护的 origin 信息泄露给不安全的域。

![unsafe url](/images/unsafe_url.png)

### no-referrer-when-downgrade

这个策略是浏览器的默认策略，表示受 TLS 保护的 origin，在访问非 TLS 包含的资源时，会去掉 referrer 信息，反过来访问相同安全级别的域时 referrer 会带上 host、port、path。

![default iframe policy](/images/default_iframe_referrer_policy.png)


以上示例的项目地址:

https://github.com/fatelei/iframe-security

### 参考

- https://developer.mozilla.org/en/docs/Web/HTML/Element/iframe
- https://w3c.github.io/webappsec-referrer-policy/#referrer-policies
