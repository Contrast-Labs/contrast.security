+++
author = ["dani-litovsky-alcala"]   
title = "Contrast Agent Discovers Reflected XSS Vulnerability in Quart Framework"   
date = "2022-03-23"
description = "Dani Litovsky Alcala discusses how she identified a Cross-Site-Scripting vulnerability in a common python framework."
tags = [
    "xss",
    "python",
    "contrast-agent",
]
featured = true
thumbnail = "images/quart.png"
+++


If you love the [Flask](https://flask.palletsprojects.com/) framework but would like to leverage the async features of Python 3, [Quart](https://pgjones.gitlab.io/quart/) is a microframework à la Flask intended to transition applications from Flask to Quart.

Contrast aims to support customers and developers using any framework. As part of my work to add support for Quart, I discovered a vulnerability in Quart v0.16.3 and earlier. This vulnerability was quickly confirmed by the Contrast [Python Agent](https://pypi.org/project/contrast-agent) 

## Reflected XSS Vulnerability in Quart

Before delving into the vulnerability I found in Quart code, allow me to point you to some excellent resources. Reflected XSS is a well documented vulnerability. You may find more information in our Contrast Security [Cross-Site Scripting](https://www.contrastsecurity.com/knowledge-hub/glossary/cross-site-scripting-prevention) page which describes various types of XSS vulnerabilities, including Reflected XSS, and the [OWASP Foundation](https://owasp.org/www-community/attacks/xss/).

As a Python Agent engineer, I'm deeply familiar with all types of security vulnerabilities. As I worked on adding Contrast support for Quart, I noticed that the `redirect` function [injected](https://gitlab.com/pgjones/quart/-/blob/0.16.3/src/quart/utils.py#L37) the location of the redirected url directly in the response body. I saw no sanitization of the `location` argument. I dug around the public [Quart documentation](https://pgjones.gitlab.io/quart/tutorials/blog_tutorial.html) and noticed that all calls to `redirect` relied on `url_for` having been called. My senses began tingling:  we can't always expect programmers will properlly sanitize and encode input. I could see that anything passed on to `redirect` would be reflected right back to the user via the response body.

To test my hypothesis, I ran the Contrast Python Agent and manually verified what I saw with my own eyes with a small Quart app.

```
from quart import Quart, request, redirect
from contrast.quart import ContrastMiddleware

app = Quart(__name__)
app.asgi_app = ContrastMiddleware(app)

@app.route("/bad")
async def unvalidated_redirect():
    user_input = request.args.get("user_input")
    return redirect(user_input)
```

Serving the app with the ContrastMiddleware means that the Python Agent will assess the application code. The Python Agent immediately alerted me to a Reflected XSS vulnerability on the this app.

Please note that Python Agent support for Quart will be released soon. You will be able to reproduce this at that time. To learn how to use the Python Agent for currently supported frameworks, please visit our [documentation](https://docs.contrastsecurity.com/en/python.html).

To manually verify that any input I send as a `user_input` parameter will be reflected back in the response body, I issued this command on the terminal

`curl "http://localhost:8000/bad?user_input=<script>alert(1)</script>"`

It responded back with

```
<!doctype html>
<title>Redirect</title>
<h1>Redirect</h1>
You should be redirected to <a href="<script>alert(1)</script>"><script>alert(1)</script></a>, if not please click the link
```

Voila! The `user_input` script is right there. For more proof, I opened this on a browser.

<img src='../../images/reflected-xss.png' />


At Contrast we value teamwork, so my next step was to ask my fellow Python Agent developers to verify what I saw, and they confirmed. The Contrast Labs team jumped on and also confirmed the finding.


## Response from Quart Maintainer

Once Contrast confirmed my finding, I followed Contrast's [Vulnerability Disclosure Policy](https://www.contrastsecurity.com/disclosure-policy) and reached out to Quart's maintainer [Phil Jones](https://pgjones.dev/) via email. He responded briefly after and agreed I found a security bug that needed to be fixed ASAP.

Phil issued a [fix](https://gitlab.com/pgjones/quart/-/commit/d125044a2292231fc5edd95cc5110d08640ecab5) to the Quart code. I hope it gets released quickly!
