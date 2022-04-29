# Convert_IE_to_Chrome

## IE 可以用name 取值

```html
<input name="test" vvalue="val1">


<script type="text/javascript">
    alert(test.value);
</script>
```

chrome 必須設定 `id`

```html
<input id="test" name="test" vvalue="val1">


<script type="text/javascript">
    alert(test.value);
</script>
```

## funcion 名稱 不可以與 元素名稱相同

當 funcion 名稱 與 元素名稱相同時 瀏覽器會找不到 funcion

```html

<input type='button' name='QueryTax' value='取財稅資料(T)' accessKey='T' onClick='QueryTax()'><BR />

<script type="text/javascript">
    function QueryTax(){
        ....
    }
</script>

```

## funcion 名稱 不可以與 元素 `id` 相同

當 funcion 名稱 與 元素名稱相同時 瀏覽器會找不到 funcion

```html

<input type='button' id='QueryTax' value='取財稅資料(T)' accessKey='T' onClick='QueryTax()'><BR />

<script type="text/javascript">
    function QueryTax(){
        ....
    }
</script>

```

## 各瀏覽器中的支援不同

`document.all`

這個會遍歷頁面所也的元素

* 替代方式

```javascript
let objs = document.getElementsByTagName('*');
```

[document.all 在各瀏覽器中的支援不同](https://www.796t.com/content/1546462261.html)

[[筆記] 使用標準 HTML DOM 物件取代 document.all 集合](https://jumping-fun.blogspot.com/2013/12/html-dom-instead-of-document.all.html)

## IE 獨有

## <col style="display:none" > 標籤

<col style="display:none" > 在 IE 少可以將 標注欄位不顯示，但是在 Chrome 不可以

```html
<table>
    <colgroup>
        <col width="30" align="center">
        
        <col style="display:none" width="270" align="center">
        
        <col width="85" align="center">
    </colgroup>
</table>
```

### 解決方式

```html
<style type="text/css">
    #grdBltList2 tr td:nth-child(2){
        display: none;
    }
    #grdBltList2 tr th:nth-child(2){
        display: none;
    }
</style>
```

## Date().getYear()

```javascript
var df = new Date();
df.getYear();

// IE: >> 當下西元年 : 2022
// Chrome >> 122

```

### 解決方式

* getFullYear

```javascript
var df = new Date();
df.getFullYear();
```

## parent.navigate 更改 瀏覽器的網址列

```javascript
parent.navigate(yourUrl);
```


### 解決方式

```javascript
parent.location.href = yourUrl;
```

## 

## 參考資料

[为什么在<col style="display:none">不起作用](https://zhidao.baidu.com/question/1174360040437688979.html)

[IE、FF、Chrome瀏覽器中的JS差異介紹](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/289453/)