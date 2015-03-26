---
module: i-parcel
maintainer: Marco Asbreuk
title: unescape template-content
intro: ""
includeexample: 20em
---

<p>Code-example:</p>


```html
<body>
    <i-parcel lazybind="true">
    <!--
        Escaped will look like: <%= what %>.
        <br>
        Unescaped will look like: <%=raw what %>.
    -->
    </i-parcel>
</body>
```

```js
<script src="itagsbuild-min.js"></script>
<script>
    require('itags');
    var iparcel = document.getElement('i-parcel'),
        model = {what: 'a <b>template</b>'};
    iparcel.bindModel(model);
</script>
```