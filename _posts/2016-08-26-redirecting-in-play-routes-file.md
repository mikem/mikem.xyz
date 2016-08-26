---
layout: post
title: Redirecting in the Play routes File
---

In a Play application I recently found myself wanting to redirect `GET /` to `GET /elsewhere`. I could have mapped `GET /` to a real controller which always performs the same redirect. But do I have to?

It turns out that Play has a `Default` controller[^1] which has some useful methods:

* `error` returns a 500 Internal Server Error
* `notFound` returns a 404 Not Found
* `redirect` takes a string argument and redirects there with a 303 See Other
* `todo` returns a 501 Not Implemented

In my case I have this in my `routes` file:

```scala
GET  /           controllers.Default.redirect(to = "/elsewhere")
GET  /elsewhere  controllers.ElsewhereController.index
```

Now `GET /` requests are redirected:

```
$ curl -I localhost:9000
HTTP/1.1 303 See Other
Location: /elsewhere
Content-Length: 0
Date: Fri, 26 Aug 2016 12:21:52 GMT
```

[^1]: Check out [the API documentation](https://www.playframework.com/documentation/2.5.6/api/scala/index.html#controllers.Default$) for more details. To see the four methods I mention, set the "Inherited" section to "Hide All" and select both "Default" options.
