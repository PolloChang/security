# CSP導入研究

##### tags: `CSP` `資安`

## 概要

CSP 全稱為 Content-Security-Policy(內容安全策略)，是X-Frame-Options 的加強板，因為配置需要花較多心思處理，所以這邊著重在配置。

下面的配置僅為經驗，當作參考，每個網站的配置不一定都相同。

## 檢測及開發工具

* CSP Evaluator

[CSP Evaluator](https://csp-evaluator.withgoogle.com/)可以檢測是否設定成功及開發 CSP

* 開發環境配置 `haproxy` 或是 `nginx`

這邊強烈建議在開發環境中自行建立 haproxy 或是 nginx 服務，因為在設定 CSP 過程中非常容易造成服務異常

## 事前準備

### 產出 nonce 值

以linux 為例，可以使用`sha256sum`產出

```bash
❯ echo -n "password" | sha256sum
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8  -
```

取其中一段字就可以了，例如： 047151d0e56f 。之後 `haproxy` 或是 `nginx` 與 網站程式碼 都需要用相同的 `nonce值`

### 調查會引入網站資源的 URL

為了防止跨網域攻擊，CSP 採用嚴謹的白名單配置。所以請先調查設定目標網展有引用哪些外部網站資源，外部資源如： `https://www.google-analytics.com`  等等

## 執行導入

導入方面需要從兩個方向進行，

1. HTTP 的 header 需要有 CSP 相關設定，這部份可以從nginx、haporxy 等負載平衡器進行處理。
2. 檢查程式碼：
   1. 在HTML 所有的 `<script>` 新增nonce
   2. 移除在HTML標籤中執行的 javascript 的程式

### HTTP header 設定

#### nginx 設定

* nginx.confg

```confg
add_header Content-Security-Policy "script-src 'unsafe-inline' 'strict-dynamic' 'nonce-047151d0e56f' https://domain1 https://domain2 ;object-src 'none';base-uri 'none';"
```

#### haproxy 設定

* haproxy.cfg

```confg
http-response set-header Content-Security-Policy "script-src 'unsafe-inline' 'strict-dynamic' 'nonce-047151d0e56f' 'unsafe-inline' https://domain1 https://domain2";"object-src 'none'";"base-uri 'none'";

```

### 前端HTML 設定原則

1. 在網站專案中所有的`<script>`、`<style>` 都需要加入 `nonce` 元素

```html
<html>
    <head>
        <script nonce="047151d0e56f">
            ...
        </script>
        <script nonce="047151d0e56f">
            ...
        </script>
    </head>
</html>
```

3. 在HTML所有元素中,嚴格禁止執行javascript

### 前端導入遭遇的困境與處理方式

下列敘述是針對網站前端導入CSP 過程中所遇到的情況與解決方式。

#### a 作為返回歷史紀錄功能失效

* 原始出現的HTML 或 javascript 片段

```html
<a title="返回" href="javascript:history.back();">
```

* 解決方式

```html
<a title="返回" href="#">
<script id="idVal" nonce="15d6c15b0f00a08">
    document.getElementById('idVal').onclick = function () {
        window.history.back();
    }
</script>
```

#### new Function

解決方式：

1. 在HTTP Header 設定 'unsafe-eval'

```conf
script-src 'nonce-J9WcAqOyWxthweB23pB4Hw==' 'strict-dynamic' 'report-sample' 'unsafe-eval' ;
```

2. 改寫程式

這部份處理詳細請參閱下列內容:

[Not compatible with strict CSP due to new Function(..)](https://github.com/jquense/expr/issues/1)

[Content Security Polic](https://developers.google.com/web/fundamentals/security/csp#if-you-absolutely-must-use-it)

#### `<iframe>` onload

* 原始出現的HTML 或 javascript 片段

```html
<script nonce="...">
    this._contentobj.innerHTML = "<iframe style='width:100%; height:100%' frameborder='0' onload='var t = $$(this.parentNode.getAttribute(\"view_id\")); if (t) t.callEvent(\"onAfterLoad\",[]);' src='about:blank'></iframe>";
</script>
```

* 解決方式

```html
<script nonce="...">
    jQuery(function () {
        iframeFunction();
    });

    function iframeFunction(){
        let t = $$(this.parentNode.getAttribute("view_id"));
        if (t) t.callEvent("onAfterLoad",[]);
    }


    ...
    this._contentobj.innerHTML = "<iframe style='width:100%; height:100%' frameborder='0' src='about:blank' ></iframe>";
    ...
</script>
```

#### HTMLElement.click() 被禁用

* 原始出現的HTML 或 javascript 片段

```javascript
document.getElementById("defaultOpen").click();
```

* 解決方式

值接呼叫 `onclick` 事件

```javascript
    jQuery(function () {
        openCity(event, 'Tab2');
    });
```

#### HTMLElement.onchange 事件

* 原始出現的HTML 或 javascript 片段

```javascript
    /**
     * 將
     * <select onchang="..."></select>
     * 進行改寫，因為在CSP 政策中是不允許的
     */
    jQuery(document).off('change','select[data-pwntpc="onchange"]');
    jQuery(document).on('change','select[data-pwntpc="onchange"]',function(){
        let functions = this.dataset.functions;
        eval(functions);
    });
```

#### `onError` 被禁用

* 原始出現的HTML 或 javascript 片段

```html
<img src=’3' onerror=this.remove();alert(1)>→ <img src=’3'>
```

* 解決方式

```javascript
/**
 * 解決 onError:
 * <img class="thumb" alt="..." onError="..."  src="...">
 *   因為在CSP 政策中是不允許的
 data-pwntpc="imgLoadError"
*/
jQuery('img[data-pwntpc="imgLoadError"]').bind('error',function(){
    this.onerror=null;this.src='${resource(dir: 'frontPage/images', file: 'img-holder.png')}';
});
```

#### 在 form.submit 中的 javascript

* 原始出現的HTML 或 javascript 片段

```html
<form class="form-inline" onsubmit="jQuery.ajax({type:'POST',data:jQuery(this).serialize(), url:'${createLink(controller:'memb',action:'filterMoneyHistory')}',success:function(data,textStatus){jQuery('#historyMoneyResultArea').html(data);},error:function(XMLHttpRequest,textStatus,errorThrown){}});return false" >

...

<button class="btn btn-orange btn-lg" type="submit">查詢</button>

...

</form>

```

* 解決方式

```html
<html>
    <head>
        <script nonce="....">
            /**
             * 將
             * <button onclick="functionName(...);" >...</button>
             * 進行改寫，因為在CSP 政策中是不允許的
             * dataSet使用範例
             <button
            type="button" data-pwntpc="callfunction" data-functions="..."
            >...</button>
            */
            jQuery(document).off('click','button[data-pwntpc="callfunction"]');
            jQuery(document).on('click','button[data-pwntpc="callfunction"]',function(){
                let functions = this.dataset.functions;
                eval(functions);
            });

            /**
             * 資料查詢
             * @param url
             * @param submitId 要提交的表單 Id
             * @param replaceId 要取代的 div 元素 Id
             */
            function searchData(url,submitId,replaceId){
                jQuery.ajax({
                    type: 'POST',
                    data: jQuery('#'+submitId).serialize(),
                    url : url,
                    success:function(data,textStatus){
                    jQuery('#'+replaceId).html(data);
                    },
                    error:function(XMLHttpRequest,textStatus,errorThrown){
                    }
                });
            }
        </script>
    </head>
    <body>
        <form id="submitId" class="form-inline" action="...">
        <button class="btn btn-orange btn-lg" type="button" data-pwntpc="callfunction" data-functions="searchData('url','submitId','replaceId')">查詢</button>
        </form>
    </body>
</html>


```

## 參考資料

[Content-Security-Policy - HTTP Headers 的資安議題 (2)](https://devco.re/blog/2014/04/08/security-issues-of-http-headers-2-content-security-policy/)

[Using a nonce with CSP](https://content-security-policy.com/nonce/)

[Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

[Active Content-Security-Policy (CSP) and Rails :back link](https://stackoverflow.com/questions/38099220/active-content-security-policy-csp-and-rails-back-link)

[内容安全政策](https://developers.google.com/web/fundamentals/security/csp?hl=zh-cn)

[`onError` 被禁用: [XSS 2] 如何防禦 XSS 攻擊](https://medium.com/hannah-lin/xss-2-%E5%A6%82%E4%BD%95%E9%98%B2%E7%A6%A6-xss-%E6%94%BB%E6%93%8A-18fdf10ef5ef)

[Not compatible with strict CSP due to new Function(..)](https://github.com/jquense/expr/issues/1)

[Content Security Polic](https://developers.google.com/web/fundamentals/security/csp#if-you-absolutely-must-use-it)