---
module: i-tabpane
maintainer: Marco Asbreuk
title: i-tabpane with inner itags
intro: ""
includeexample: 20em
---

<p>Code-example:</p>


```html
<body>
    <i-tabpane pane="2">
        <!--
        <section><span is="tab">item 1</span>Content first page</section>
        <section><span is="tab">item 2</span>Content second page</section>
        <section><span is="tab">item3 header</span>page 3</section>
        <section>
            <i-form>
                <!==
                    <i-select value="2" class="i-primary">
                        <!==
                            <option>item1</option>
                            <option>item2</option>
                            <option><span is="button">item3 header</span>item3 content</option>
                            <option>item4</option>
                            <option>item5</option>
                        ==>
                    </i-select>
                    <i-input reset-value="nobody" placeholder="Say your name"></i-input>
                ==>
            </i-form>
        </section>
        <section><i-input id="2" reset-value="nobody" placeholder="Say your name">Marco</i-input></section>
        -->
    </i-tabpane>
</body>
```

```js
<script src="itagsbuild-min.js"></script>
<script>
    var model = {
        value: 4,
        'primary-button': true
    };
    var inputmodel = {
        value: 'Harry'
    };
    document.bindModel(model, 'i-select');
    document.bindModel(inputmodel, 'i-input');
    document.getElement('i-tabpane').focus();

</script>
```