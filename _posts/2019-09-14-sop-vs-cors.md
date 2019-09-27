---
title: The Browsers Yin and Yang | Security Considerations on SOP and CORS
---

>"In Chinese philosophy, yin and yang (/jɪn/ and /jɑːŋ, jæŋ/; Chinese: 陰陽 yīnyáng, lit. "dark-bright", "negative-positive") is a concept of dualism in ancient Chinese philosophy, describing how seemingly opposite or contrary forces may actually be complementary, interconnected, and interdependent in the natural world, and how they may give rise to each other as they interrelate to one another."



First of all, lets start out with specifying base concepts.


## What is Same-Origin-Policy?

The same-origin policy is a critical security mechanism that restricts how a document or script loaded from one origin can interact with a resource from another origin.



## What is an origin?

Two URLs have the same origin if the protocol, port (if specified), and host are the same for both. This means that if two URL is using
###
- **https://** scheme
- both have the same host like **mail.google.com**
- operating on the same port like **:443** 

these two URLs have the same origin. 


![image](/img/origins.png)

There are some rules for changing origin from a subdomain to its parent domain, E.g: mail.google.com to google.com. To be able to change the origin, it is required to set document.domain to same value in both parent domain and subdomain. However, it is not allowed to change domain from google.com to yahoo.com. For more information, see [document.domain.](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain)


## Why, Same-Origin-Policy?

HTTP requests that are sent from one origin to a different one are called cross-origin requests. SOP was implemented to prevent this kind of cross-origin requests because of potential security flaws like [CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)), [Clickjacking](https://www.owasp.org/index.php/Clickjacking) etc. However, as web applications became more complicated and interconnected and tangled, need for cross-origin requests initiated by JavaScript became a necessity. To loosen the rules layed by Same-Origin-Policy and to enable cross-origin requests, standard for Cross-Origin Resource Sharing(CORS) was introduced.

## How to make a CORS Request? More importantly, Why make a CORS Request?

As for why, it is pretty simple. Let's say that you want to use a CDN tp boost your website performance. If you are using a CDN like Cloudflare, your site needs to load some assets(images, css files, etc.) externally to take the load off of your servers. In order to do this, website needs to be able to send a cross-origin request to Cloudflare to retrieve the assets your web page needs.


As for how, CORS standard says that there are 2 type of requests when dealing with cross-origin requests. First one is called simple request. If;

###
- HTTP method is GET/HEAD/POST
- Request contains standard headers like Accept, Accept Language, Content-Language, Content-Type, for more information on these headers you can check [here.](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)
- Request submits content type as "application/x-www-form-urlencoded","multipart/form-data" or "text/plain" (types that can be submitted by HTML forms)

Request is considered simple and If the request is simple, browser can send the request to an external origin, but target's CORS policy must allow the request as well. If the request is not simple, browser must do a preflight request with OPTIONS method and appropriate CORS headers to negotiate transmission rules. If target's CORS policy does not allow the request, then browser must refuse to send the actual request. Targets should respond to a cross-origin request with the following headers:

###
- Access-Control-Allow-Origin: either a single allowed origin or a wildcard (*) indicating all origins; servers may change the value depending on the Origin of the request
- Access-Control-Allow-Credentials: indicating if the browser is allowed to send cookies with the request; if omitted, defaults to false; cannot be true if Access-Control-Allow-Origin is *
- Access-Control-Allow-Headers: comma-separated list of allowed headers

Now I know this is a bit confusing, think of it like a TCP handshake. Browser and the server are trying to negotiate on the terms of how to share resources. Here is a simple flowchart of this process, taken from Wikipedia. Think of green zone request as simple request, red zone request as preflight requests.

![image](/img/simple-preflight-requests.png)


An even simpler one, taken from AWS docs.

![image](/img/cors-simple-request.png)

Preflight checks by browser and SOP make sure that HTTP moethods which may modify sensitive data, such as DELETE, PUT, PATCH and non-simple POST requests will never be sent to the server from a third-party domain, unless the sever explicitly allows such request. However, GET requests usually gets past this security consideration. Let's imagine a bank, which an online banking system uses get requests to transfer money between accounts like this;

https://bankofironislands.com/moneyTransfer?fromAcc=123&toAcc=456&Amount=500

This is a classic CSRF example, which is a very dangerous practice even without considering SOP implementations and there are not many systems using GET requests to do important data transaction(hopefully) but for the sake of example, let's examine it. As far as the browser is concerned, there is nothing wrong this this type of request because it is a legitimate request which uses redirection to another domain. From bank's perspective, this is also a legitimate request without any malicious content. Attackers could also leverage embedding 3rd party content to their web sites to form a clickjacking attack or use an HTML form to send a GET request to the bank. SOP, as it is, can not do anything to prevent these kind of situations when dealing with GET requests and this is exactly why [RFC](https://tools.ietf.org/html/rfc7231#section-4.2.1) states that GET requests should never change web application data. Unfortunately, some sites still use GET requests for important data and data modifying purposes, which makes attackers job easy.
