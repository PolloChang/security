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

## IE 獨有

`document.all`

這個會遍歷頁面所也的元素

* 替代方式

```javascript
let objs = document.getElementsByTagName('input');
```

[IE、FF、Chrome瀏覽器中的JS差異介紹](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/289453/)