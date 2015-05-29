---
module: i-splitdiv
maintainer: Marco Asbreuk
title: changed by model
intro: ""
includeexample: 20em
---

<p>Code-example:</p>

```css
<style type="text/css">
    i-splitdiv {
        width: 500px;
        height: 190px;
        position: absolute;
        top: 50px;
        left: 50px;
        border: solid 1px #000;
    }
    i-splitdiv section {
        background-color: #DDD;
        padding: 20px;
    }
    i-splitdiv section[section="second"] {
        border-left: dotted 4px #000;
        background-color: #EEE;
    }
</style>
```

```html
<body>
    <i-splitdiv vertical="true" divider="20px">
    <!--
        <section>item1</section>
        <section>item2</section>
    -->
    </i-splitdiv>
</body>
```

```js
<script src="itagsbuild-min.js"></script>
<script>
    require('itags');
    ITSA.later(function() {
        document.getElement('i-splitdiv').model.divider = '125px';
    }, 1000);
</script>
```