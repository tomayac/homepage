Title: On the X-Frame-Options Security Header
Date: 2013-12-12
Slug: x-frame-options-security-header
Author: Frederik

This blog post *about X-Frame-Options* was originally published on the [Mozilla Security Blog](https://blog.mozilla.org/security/2013/12/12/on-the-x-frame-options-security-header/)

A few weeks ago, [Mario Heiderich](http://heideri.ch/) and I published a [white paper](https://frederik-braun.com/xfo-clickjacking.pdf "X-Frame-Options: All about Clickjacking?") about the [X-Frame-Options security header](http://tools.ietf.org/html/rfc7034 "RFC7034"). In this blog post, I want to summarize the key arguments for settings this security header in your web application.

X-Frame-Options is an optional HTTP response header that was introduced in 2008 and found its first implementation in Internet Explorer 8. Setting this header in your web application defines if it works within a frame element (e.g., `iframe`). The syntax for this header provides three options, ALLOW-FROM, DENY or SAMEORIGIN. Not sending this header implies allowing frames in general. ALLOW-FROM, however, allows whitelisting a specific [origin](http://tools.ietf.org/html/rfc6454#section-3.2.1 "An origin is the scheme, host and the (optional) port of a URI"). The opposite is, of course, DENY which means that no website is ever allowed to display your website in a frame. A common middle ground is to send SAMEORIGIN. This means that only websites of the [same origin](http://tools.ietf.org/html/rfc6454#section-3.2.1 "Being same-origin means having the same scheme, host and port") may frame it.

This blog post will highlight some attacks than can be thwarted by forbidding the framing of your document. First of all, **Clickjacking**. This term has gained major attention in 2008 and includes a multitude of techniques in which an evil web page can secretly include yours in a frame. But the author of this evil website will make your website transparent and present buttons on top of it. Anyone visiting this evil page will then click on something seemingly unrelated, which will actually result in mouse clicks in your web application.

A wide class of attacks on other websites leverage missing security features in the browser. Most modern browsers provide hardened security mechanisms that may easily thwart problems with content injections. The problem lies, as so often, in [backwards compatibility.](http://msdn.microsoft.com/en-us/library/jj676915%28v=vs.85%29.aspx "Microsoft explains legacy document modes") The most recent browser versions are obviously more secure than the previous ones. But when somebody frames your website, they can tell it to run in a **compatibility mode**. This feature only applies to Internet Explorer, but it will bring back the vintage rendering algorithms from IE7 (2006). In Internet Explorer, the _document mode_ is inherited from the top window to all frames. If the evil websites runs in IE7 compatibility mode, then so does yours! This is an example of how IE7 compatibility can be triggered in any website:

`<meta http-equiv="X-UA-Compatible" content="IE=7" />`

If your website would not allow to be framed, your IE users were not at risk.

Another technique for possible attackers comes with **window.name**. This attribute of your browsing window (a tab, a popup and a frame are all windows in JavaScript's sense) **can be set by others** and you cannot prevent it. The implications of this are manifold, but just for the sake of [**Cross-Site-Scripting** (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting "as explained in the Wikipedia") attacks it may make things for an attacker much easier. Sometimes, when an attacker is able to inject and execute scripts on your web page, he might be hindered by a length restriction. Say, for example, your website does not allow names that exceed 80 characters. Or messages that must not exceed 140. The window.name property can help bypassing these restriction in a very easy way. The attacker can just [frame your website ](# "I hope you notice a pattern here")and give it a name of his liking, by supplying it in the frame's `name` attribute. The JavaScript he will then execute can be as short as `<svg/onload=eval(name)>`, which means that it will execute the JavaScript specified in the name attribute of the frame element.

These and [many other attacks](https://frederik-braun.com/xfo-clickjacking.pdf "Interested? Read the full document.") are possible if you allow your web page to be displayed in a frame. Just recently, Isaac Dawson from Veracode has published a [report about security headers](http://www.veracode.com/blog/2013/11/security-headers-on-the-top-1000000-websites-november-2013-report/) on the top 1 million websites, which shows, that only 30,000 of them currently supply this header. However, the fact that many other sites are vulnerable to these sort of attacks is not a good reason to leave your website unprotected. You can easily address many security problems by just adding this simple header to your web application right away: If you're using Django, check out the [XFrameOptionsMiddleware](https://docs.djangoproject.com/en/dev/ref/clickjacking/# "Django X-Frame-Options"). For NodeJS applications, you can use the [helmet library](https://npmjs.org/package/helmet) to add security headers. If you want to set this header directly from within Apache or nginx, just take a look at the [X-Frame-Options article on MDN](https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options).