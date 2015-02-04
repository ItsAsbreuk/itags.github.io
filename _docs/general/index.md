---
module: general
itsaclassname:
version: 0.0.1
modulesize: 1.69
dependencies: "js-ext, io, polyfill, utils, vdom, event-dom"
maintainer: Marco Asbreuk
title: Itag Classes
intro: "Defining Itag Classes to create itag - custom elements"
firstpar: get-started
---



#The Basics#

The dom has a limited number of Elements like `div`, `input`, `button` etc. Custom Elements are a way to extend these with your own custom element-name [more info here](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)

`Itags` is our solution into creating custom elements. To create `Itags`, we developed an api based upon Classes. All `Itag`-elements need to start with `i-` followed by a name.

Every `Itag` needs to be defined by its own module. Once this module is loaded, two things happen:

* all elements on the page that are of the specified itag-type get upgraded into a true itag
* document.createElement() can accept the itag-name and will create a true itag-element


####Example i-select####

```html
<i-select value="2">
    <i-item>item 1</i-item>
    <i-item>item 2</i-item> <!-- is the selected item -->
    <i-item>item 3</i-item>
    <i-item>item 4</i-item>
</i-select>
```

Itags are special elements, because they have dedicated features:

##Features##

* Inner nodes are not accessible through javascript (except from i-parcel)
* Unidirect dataflow: element.model is leading
* Dom events on innernodes will not reach out if the itag (except from i-parcel)

##Structure##

`Itags` are HTMLElements with a custom innerHTML which is defined by its Class-definition. The innerHTML is hidden from any node-query, except that they can be queried from within itself. When initialized, any pre-set innerHTML can be used to set up the element-data, which is available at element.model {Object}. In most cases, the preset innerHTML is used together with attribute-data to build up the initial element.model. Once intialized, the preset innerHTML will be removed automaticly (irrelevant if it's used) and a new innerHTML will be rendered by the itag-instance. The new innerHTML can be rendered in both the initializer (which is available is the **init()** method) and/or the **sync()** method. The difference is that the sync-method gets invoked everytime element.model changes.

Itags have their state present on a property `model`, which is an object which is available for every itag-instance (from now on we will refer to it as: `element.model`). Element.model determines in a uniflow structure the appearance of the itag. Any UI-interaction should be stored inside element.model.someproperty, which will lead to re-syncing and matching the UI. Any itag can use some (or all) of its attribute to be bound to element.model.attributeName as well. The attributes that should be bound can be specified inside the Itag-Class definition by the prototype-property `attrs` (more on this later).

The `sync`-method should be used to re-set any innerNodes. Because `itags` are using the module `itsa/vdom`, syncing will de done using the virtual dom (diffing).

`Itags` remain up to date by either one of these 2 procedures:

###Object.observe###
`Object.observe` on element.model is the prefered way of keeping the itags up to date. At this moment only supported by Chrome and Opera. Whenever element.model changes, the according itag-instance will update its relevant attributes and will invoke its sync-method.

###Finilazer###
For browsers that do not support this `Object.observe`, we make use of a special feature: a `finilizer`. What happens is that after every eventcycle (either Event, XHR or setTimeout) all itags will refresh in a same way that Object.observe did for its own instance. This may seem heavy, but refreshing is done by the vdom using diffing: nothing will change if there is nothing to change. We choosed this way of keeping up to date, because it is preferable above polling for changes in the modeldata (which could consist a huge numer of objects considering large arrays). There is no wasted polling and there won't be any missing updates: <u>changes in modeldata don't come out of the blue</u> --> they always happen after the eventcycle needed to perform some action.

Some dom-events are likely to not effect any element.model data. To enlighten the system, these events do not lead into a direct resyncing of the itags, but are delayed to happen only once a second. Read more about these events (and how to make them directly resync) here..



#Developing a new Itag#

`Itags` are `Classes`, having the API that we created for using Classes (more about that later). Itags should be developed in a module of their own. This way, you are sure the right dependencies are loaded: you need at least the `itags.core` module. Also, distribution and further dependiencies are made easy this way. Once this module is loaded, all elements in the dom of the type `i-yourname` get upgraded (as said before).

There are three different ways of developing an Itag-Class:

* create completely new using `createItag()`
* sub-classing an existing Itag-Class with `subClass()`
* pseudoClassing an existing Itag-Class with `pseudoClass()`

In either case, the basic skeleton look like this (example is based upon `createItag`):

####default skeleton defining itags####
```js
module.exports = function (window) {
    "use strict";

    require('polyfill/polyfill-base.js');

    var itagCore =  require('itags.core')(window),
        itagName = 'i-yourname', // <-- define your own itag-name here
        DOCUMENT = window.document,
        Itag;

    if (!window.ITAGS[itagName]) {

        Itag = DOCUMENT.createItag(itagName, {
            attrs: {
            },

            init: function() {
            },

            sync: function() {
            },

            destroy: function() {
            }
        });

        window.ITAGS[itagName] = Itag;
    }

    return window.ITAGS[itagName];
};
```

*Note:* every new definition is automaticly stored inside the object `window.ITAGS` like this: *window.ITAGS[itagName] = ItagClass;*


##createItag##

You can use `window.document.createItag()` for every new itag you want to setup that doesn't need to be inherited from an existing itag. Probably most of the cases. The first argument holds the *name* of the `itag`. The second argument (as well as the other) are discussed later on.

####Example document.createItag()####
```js
document.createItag('i-myform');
```


##subClass##

Any `Itag-Class` can be subclassed with 'subClass()`. The subclass has its own itag-name and its own members (second argument), but more about that later. By default, the `init`-method of the inherited Class gets invoked before `init` of the subclassed Itag.

Important (and different compared to `pseudoClass`) is that the itag <u>has its own element-name</u>. This is important, because you probably loose `css`-styles and maybe events (when they are filtered by element-name as they should) by the superclass. So, you need to redefine them again.

Subclassing using `subClass` isn't very useful unless you want to inherit a lot of the functionality that was defined by its parent by the second argument (member of the prototype). Mostly you be better of using `document.createItag()` or `ItagClass.pseudoClass()`.


####Example defining subClass####
```js
var MyIFormClass, AnotherFormClass;

MyIFormClass = window.ITAGS['i-myform'];
AnotherFormClass = MyIFormClass.subClass('i-anotherform');
```


##pseudoClass##

Any `Itag-Class` can also be subclassed with 'pseudoClass()`. As its first argument you pass the `pseudo`-name. Registering need to be done by a composite name: the subclassed its itagname, followed by a `hashtag` (#), followed by the pseudoname.

####Example defining pseudoClass####
```js
var MyIFormClass, AnotherFormClass;

MyIFormClass = window.ITAGS['i-myform'];
AnotherFormClass = MyIFormClass.pseudoClass('anotherform');
```

Elements created by `pseudoClass` will render with the same element-name as its parent-Class. The `pseudo`-name will be set at the `is`-attribute:

####Example pseudoClass html-element####

```html
<!-- the next element will be rendered as an itag of the Class: window.ITAGS['i-myform#anotherform'] -->
<i-myform is="anotherform"></i-myform>
```

####Example pseudoClass html-element by javascript####

```js
var element = document.createElement('i-myform#anotherform');
document.body.appendChild(element);

// this adds an new element looking like: <i-myform is="anotherform"></i-myform>
```

The beautiful part here is, that any styles and events of the superclass are retained.


##Passing arguments##

In the previous methods to create itags, you can pass 5 arguments:

* itagname/pseudoname
* prototypes
* chainInit (not applyable for createItag)
* chainDestroy (not applyable for createItag)
* subClassable

###itagname/pseudoname###

The first argument defines the name of the itag-element. This has to be a String starting with "**i-**". In case of pseudo-classing, it must be a string without a minustoken, leadin into the definition of an element like "i-parent#pseudo"

###prototypes###

This is an optional object containing all members (properties and methods) that should be available on the prototype of the defined itag. There are 4 special members that need attention, because they have special behaviour which make itags work very expressive:

* **attrs** {Object} You can specify which attributes are bound to element.model by defining the `attrs`-object. Every item has a name (attributename) and a String-value defining one of these types: "String", "Boolean", "Number", "Date" (case-insensitive). All attributes defnied here will be life synced with element.model.

####Example defining attrs####

```js
var MyIFormClass = document.createItag('i-myform', {
    attrs: {
        action: 'string'
    }
});
```

* **init** {Function} the initialisation method: this method gets invoked once on creation of the itag. This is the place to setup. You might create an itag that reads (and destroys) innerHTML and put it into element.model.someporperty. Needs no returnvalue.

* **sync** {Function} This is the method that gets invoked on every element.model change. You should process element.model data and update parts of the element.innerHTML here. Needs no returnvalue.
Note: within the sync-method, you can set any attribute on the itag-element <u>except</u> those defined by `attrs` --> the attributes are bound to element.model and are life-updated by themselves.

* **destroy** {Function} Is invoked when the itag gets removed from the dom. You could do some cleanup here. No need to cleanup eventlisteners which are bound to the instance: this is done automaticly. Needs no returnvalue.

        * @param pseudo {String} The pseudoname (without a minustoken), leading into the definition of `i-parent#pseudo`
        * @param [prototypes] {Object} Hash map of properties to be added to the prototype of the new class.
        * @param [chainInit=true] {Boolean} Whether -during instance creation- to automaticly construct in the complete hierarchy with the given constructor arguments.
        * @param [chainDestroy=true] {Boolean} Whether -when the Element gets out if the DOM- to automaticly destroy in the complete hierarchy.
        * @param [subClassable=true] {Boolean} whether the Class is subclassable. Can only be set to false on ItagClasses


###chainInit###

This is a boolean (defaults true) which specifies that the parent Class its `init`-method should be invoked before its own `init`. This is normal behaviour when sub-classing, though can suppress it. When suppressed, you still can invoke the parent Class its `init`, by using **this.$superProp('init')**

chainInit is not applyable on document.createItag().


###chainDestroy###

This is a boolean (defaults true) which specifies that the parent Class its `destroy`-method should be invoked after its own `destroy`. This is normal behaviour when sub-classing, though can suppress it. When suppressed, you still can invoke the parent Class its `destroy`, by using **this.$superProp('destroy')**

chainDestroy is not applyable on document.createItag().


###subClassable###

This is a boolean (default true) that specifies whether the itag can be sub-classed.



#Access to super-Class properties#

##access parent properties##

When subClassing, it is easy to access properties of its parent by invoke `this.$superProp(propertyName, args)`. Any property can be invoked: when it's a method, you can pass through its arguments as from the second argument-position. <u>`$superProp` is avialabe on the context `"this"`</u>.

When a `constructor` needs to be subClassed, you can use: `this.$superProp('constructor', args)`. Be sure you set the firth argument `false` in order to be able to manually invoke the super-constructor.

####Example redefine sync####
```js
var MyIFormClass, AnotherFormClass;

MyIFormClass = window.ITAGS['i-myform'];
AnotherFormClass = MyIFormClass.subClass('i-anotherform', {
    sync: function() {
        var element = this;
        element.$superProp('sync');
        if (element.model.age>80) {
            element.append('<br><br>Don\'t rush, I\'m aged...'');
        }
    }
});
```

##access ancestor properties##

If you want to access properties that lie higher in the Class-tree (higer than `parent`), you can use `this.$super.$superProp()` or multiple `$super` parts.  <u>`$super` is avialabe on the context `"this"`</u>.

####Example redefine properties higher up the chain####
```js
var AnotherFormClass, SecondOtherFormClass;

AnotherFormClass = window.ITAGS['i-anotherform'];
SecondOtherFormClass = AnotherFormClass.subClass('i-secondotherform', {
    sync: function() {
        var element = this;
        element.$super.$superProp('sync');
        if (element.model.age>80) {
            element.append('<br><br>Don\'t need to rush, take your time...'');
        }
    }
});
```



#Reconfigure Classes#

Existing Classes cannot have their inherited (parent) Class being redefined (just define a new Class in those cases). However, they can have their prototype-properties being redefined, extended, or removed.


##mergePrototypes##

It allows to add extra methods or properties to a given Itag-Class.  This is helpful when common functionality needs to be added to multiple classes, without having to inherit from it.  For example, the previous example could have been made like this:

####Example mergePrototypes####

```js
var MyIFormClass = window.ITAGS['i-myform'];
var changeStatus = {
    enable: function () {
        this.model.enabled = true;
    },
    disable: function (x) {
        this.model.enabled = false;
    }
};

MyIFormClass.mergePrototypes(changeStatus);
```

The merged methods will not overwrite existing methods unless the second argument is set to `true` to force the overwrite.

##Using $orig in mergePrototypes##

If the merged methods override existing ones, the original method will be available in the `$orig` property, <u>which is availabe on the context `"this"`</u>.  This allows plugins that can be refer to the original methods. All arguments you pass into `$orig()` will be passed through to its original method.

It is possible to redefine the same method in descendent subClasses by using $orig() over and over again. All original methods will be available.

####Example mergePrototypes with usage $orig()####
```js
var MyIFormClass = window.ITAGS['i-myform'];
var changeStatus = {
    enable: function () {
        this.model.enabled = true;
    },
    disable: function (x) {
        this.model.enabled = false;
    },
    sync: function() {
        var element = this,
            sendButton = element.getElement('button[type="submit"]');
        element.$orig();
        sendButton.toggleClass('pure-button-disabled', !element.model.enabled);
    }
};

MyIFormClass.mergePrototypes(changeStatus, true);
```

##removePrototypes##

Existsing methods or properties at the prototype of an Itag, can be removed with `removePrototypes`:
####Example removePrototypes####

```js
var MyIFormClass = window.ITAGS['i-myform'];

MyIFormClass.removePrototypes('disable');
```


#Destroying Itag-instances#

Itag-instances are HTMLElements. They are created when inserted in the dom, or with document.createElement() and <u>are destroyed automaticly when removed from the dom</u>. Once destoyed, the itag-element cannot be used anymore. On destruction, the `destroy()`-method of the Itag (and its super-classes) get invoked. By default, these methods are empty, but they could be set upt to do some cleanign stuff. Also, alle events that are set on the element-instance are automaticly detached.

This leads into a behaviour where the elements don't leak memory while the developer doesn't need to worry about their removal.

In most cases, you don't need to setup `destroy`. Only when you have set data by closure outside the instance (for example in an array), then you need to clean it up: otherwise there would be a memoryleak. Another feature would be when the class-instantiation would create a dom-node <i>outside its own element</i>, which you need to remove at destruction. Creating domnodes outside its own element is a practice that is not recomended.

Under the hood, `destroy` gets stored as `_destroy`, whereas `Class.destroy(notChained)` is a method on the BaseClass at the highest position of the Class-chain --> this `destroy()` invokes `_destroy` of the whole chain.

####Example using destroy####

```js
var regArray = [],
    MyIFormClass, newNode;

MyIFormClass = document.createItag('i-myform', {
    init: function() {
        var element = this;
        regArray.push(element);
    },
    destroy: function () {
        var element = this;
        delete regData[element];
    }
});

newNode = document.body.append('<i-myform></i-myform>');
// regArray.length === 1

newNode.remove(); // removes the node from the dom, leading to invoke `destroy()`
// regArray.length === 0

```

Note that regArray.length gets its updated value in the next event-cycle, so checking synchronious will lead to the wrong values.



#Events#

##Event-listener##

All Itag-Classes become an Event-listener (for more info on event-listeners: see the module `Event`). This behaviour is added to the ItagBaseClass which all Classes inherit. The Event-listener makes the following properties available:

###after###
###onceAfter###
###before###
###onceBefore###
###selfAfter###
###selfOnceAfter###
###selfBefore###
###selfOnceBefore###

The methods named `selfxxx` make it posible to invoke the subscriber only when `e.target` equals the `instance`. This avoids unwanted interaction: see the examples.

##Event-emitter##

All Itag-Classes are automaticly become an Event-emitter (for more info on event-emitters: see the module `Event`). **Their emittername equals the itag-name**. Now, the elements can emit any event, which will be preceeded with the right emitterName.

####Example emitting events on an Itag-instance####

```js
var MyIFormClass, newNode;

MyIFormClass = document.createItag('i-myform');

newNode = document.body.append('<i-myform></i-myform>');

newNode.emit('send'); // will emit the `i-myform:send` event
```

####Example emitting events on an Itag-pseudo-classed-instance####

```js
var MyIFormClass, AnotherFormClass;

MyIFormClass = window.ITAGS['i-myform'];
AnotherFormClass = MyIFormClass.pseudoClass('anotherform');

newNode = document.body.append('<i-myform is="anotherform"></i-myform>');

newNode.emit('send'); // will emit the `i-myform#anotherform:send` event
```

##Detaching listers on destruction##

<u>You don't need to detach any listener you have set on any class-instance.</u>

This is done automaticly when you destroy the class by using `destroy()` - regardless of its first argument. Under the hood, `destroy()` invokes this.`detachAll()` which removes all listeners of the instance.



#Making Itag-elements focusable#

Itags-elements are unfocusable by default: the dom will only set focus on particular nodes like buttons or input-elements. Because -in the end- it is not the itag-element itself that needs the focus, but one of its rendered inner-nodes, you have to use a workarround. There are two ways to make an itag focusable, depending on the itag's features:


##Using focusmanager##
By setting `itsa/focusmanager` on the itag-element (which could be done in the init() or sync() method, depending on its nature), the itag can get focus and the focusmanager will delegate focus to one of the focusable nodes.

<ul>This is not the favored way</ul>: all logic is better keeped hidden inside the itag-element.

####Example focusmanager on Itag####

```js
document.createItag('i-myform', {
    init: function() {
        this.plug(ITSA.Plugins.focusManager);
    }
});
```

Preferable is to use an innernode by making use of the `manualfocus-event`:

##Listening for the manualfocus-event##
Another way is to listen for a `manualfocus`-event --> which gets fired before a domnode gets focussed by node.focus(). The trick is to subscribe at the before-listener, perventDefault and refocus on the node that really should get focus. Be aware though, that a page might set the focus to an itag-element while it's not rendered yet - which only happens when focus is set before the dom is ready. So, we need to wait re-setting focus until the itag-element is rendered (see next example):

####Example using the manualfocus-event####

```js
document.createItag('i-myform', {
    init: function() {
        this.append('<button>Send data</button>');
    },
});

ITSA.Event.before('i-myform:manualfocus', function(e) {
    var element = e.target;
    e.preventDefault();
    // cautious: the element might get focus before it is rendered
    // therefore, we wait until it is ready:
    element.itagReady().then(
        function() {
            var button = element.getElement('button');
            button.focus();
        }
    );
});
```

The same technique can be used in combination with the focusmanager:

####Example using the manualfocus-event with focusmanager####

```js
document.createItag('i-myform', {
    init: function() {
        this.append('<ul fm-manage="true"><input /><button>Send data</button></ul>');
    },
});

ITSA.Event.before('i-myform:manualfocus', function(e) {
    var element = e.target;
    e.preventDefault();
    // cautious: the element might get focus before it is rendered
    // therefore, we wait until it is ready:
    element.itagReady().then(
        function() {
            var ul = element.getElement('ul');
            ul.focus();
        }
    );
});
```


#Best practices#

The best way to create modules, is to start with the skeleton as mentioned above. You can erase the prototype-members you don't need (mostly <i>destroy</i>). Because itag-elements probably are interactive, you would need to setup events at the UI. These events need to update `element.model` **not** any attribute. To prevent the need of define new events for every instance, it is <u>not advised to setup event inside **init()**</u>. Instead, define these Events general using delegation: they will stay arround forever (even with no itag on the page), but you only need one to handle all itags.

##Async actions inside sync()##
**Be careful** to start async actions inside `sync()`. Make sure, that before ending, any asynchronisity gets cancelled because it will be setup over and over agian on every sync-action.


##Delayed sync-events##
Some dom-events will resync with a delay of 1 second. These events are:

* mousedown
* mouseup
* mousemove
* panmove
* panstart
* panleft
* panright
* panup
* pandown
* pinchmove
* rotatemove
* focus
* manualfocus
* keydown
* keyup
* keypress
* blur
* resize
* scroll

In case you need direct syncing from one of these events, you can specify this by using: `ItagClass.setItagDirectEventResponse(eventName)`;


##Example##

Below is the full code of an early version if `i-select`, which illustrates how an Itag-module could be set up:

####Example i-select module####
```js
require('polyfill/polyfill-base.js');
require('js-ext/lib/string.js');
require('css');
require('./css/i-select.css');

var NATIVE_OBJECT_OBSERVE = !!Object.observe,
    CLASS_ITAG_RENDERED = 'itag-rendered',
    utils = require('utils'),
    laterSilent = utils.laterSilent;

module.exports = function (window) {
    "use strict";


    var DEFAULT_INVALID_VALUE = 'choose',
        itagCore =  require('itags.core')(window),
        itagName = 'i-select',
        DOCUMENT = window.document,
        HIDDEN = 'itsa-hidden',
        SHOW = 'i-select-show',
        Event, Itag;

    if (!window.ITAGS[itagName]) {
        Event = require('event-dom')(window);
        require('focusmanager')(window);
        require('i-item')(window);
        require('i-head')(window);

        Event.before(itagName+':manualfocus', function(e) {
            // the i-select itself is unfocussable, but its button is
            // we need to patch `manualfocus`,
            // which is emitted on node.focus()
            // a focus by userinteraction will always appear on the button itself
            // so we don't bother that
            var element = e.target;
            e.preventDefault();
            // cautious: the element might get focus before it is rendered
            // therefore, we wait until it is ready:
            element.itagReady().then(
                function() {
                    var button = element.getElement('button');
                    button && button.focus();
                }
            );
        });

        Event.after('blur', function(e) {
            // the i-select itself is unfocussable, but its button is
            // we need to patch `manualfocus`,
            // which is emitted on node.focus()
            // a focus by userinteraction will always appear on the button itself
            // so we don't bother that
            var element = e.target,
                model;
            e.preventRender();
            // cautious:  all child-elements that have `manualfocus` event are
            // subscribed as well: we NEED to inspect e.target and only continue.
            //
            // I didn;t figure out why, but it seems we need `later`
            // in order to make the i-select prevent from acting unpredictable.
            // maybe because of the responsetime of the click-event
            laterSilent(function() {
                // if e.target===i-select
                if ((element.getTagName()==='I-SELECT') && !element.hasClass('focussed')) {
                    model = element.model;
                    model.expanded = false;
                    NATIVE_OBJECT_OBSERVE || DOCUMENT.refreshItags();
                }
            },350);
        }, 'i-select');

        Event.before('keydown', function(e) {
            if (e.keyCode===40) {
                // prevent minus windowscroll:
                e.preventDefaultContinue();
            }
        }, 'i-select > button');

        // CAUTIOUS: it seems `tap` will be subscribed 8 times!!!
        // TODO: figure out why not once
        Event.after(['click', 'keydown'], function(e) {
            var element = e.target.getParent(),
                expanded, value, liNodes, focusNode, model;
            if ((e.type==='click') || (e.keyCode===40)) {
                (e.keyCode===40) && e.preventDefault();
                model = element.model;
                expanded = model.expanded;
                value = element.model.value;
                if (!expanded) {
                    liNodes = element.getAll('ul[fm-manage] > li');
                    focusNode = liNodes[value-1];
                    focusNode && focusNode.focus();
                }
                model.expanded = !expanded;
            }
        }, 'i-select > button');

        // CAUTIOUS: it seems `tap` will be subscribed 8 times!!!
        // TODO: figure out why not once
        Event.after(['click', 'keypress'], function(e) {
            var liNode = e.target,
                element, index, model;
            if ((e.type==='click') || (e.keyCode===13)) {
                element = liNode.inside('i-select');
                model = element.model;
                index = liNode.getParent().getAll('li').indexOf(liNode);
                model.expanded = false;
                model.value = index+1;
                element.getElement('button').focus();
            }
        }, 'i-select ul[fm-manage] > li');

        Event.defineEvent(itagName+':valuechange')
             .unPreventable()
             .noRender();

        Event.after('*:change', function(e) {
            var element = e.target,
                prevValue = element.getData('i-select-value'),
                model = element.model,
                newValue = model.value,
                markValue;
            if (prevValue!==newValue) {
                markValue = newValue - 1;

                /**
                * Emitted when a the i-select changes its value
                *
                * @event i-select:valuechange
                * @param e {Object} eventobject including:
                * @param e.target {HtmlElement} the i-select element
                * @param e.prevValue {Number} the selected item, starting with 1
                * @param e.newValue {Number} the selected item, starting with 1
                * @param e.buttonText {String} the text that will appear on the button
                * @param e.listText {String} the text as it is in the list
                * @since 0.1
                */
                element.emit('valuechange', {
                    prevValue: prevValue,
                    newValue: newValue,
                    buttonText: model.buttonTexts[markValue] || model.items[markValue],
                    listText: model.items[markValue]
                });
            }
            element.setData('i-select-value', newValue);
        }, itagCore.itagFilter);

        Itag = DOCUMENT.createItag(itagName, {
           /**
            * Redefines the childNodes of both the vnode as well as its related dom-node. The new
            * definition replaces any previous nodes. (without touching unmodified nodes).
            *
            * Syncs the new vnode's childNodes with the dom.
            *
            * @method _setChildNodes
            * @param newVChildNodes {Array} array with vnodes which represent the new childNodes
            * @private
            * @chainable
            * @since 0.0.1
            */
            init: function() {
                var element = this,
                    itemNodes = element.getAll('>i-item'),
                    items = [],
                    buttonTexts = [],
                    content;
                itemNodes.forEach(function(node, i) {
                    var header = node.getElement('i-head');
                    if (header) {
                        buttonTexts[i] = header.getHTML();
                        header.remove(true);
                    }
                    items[items.length] = node.getHTML();
                });

                element.model.items = items;
                element.model.buttonTexts = buttonTexts;

                // store its current value, so that valueChange-event can fire:
                element.setData('i-select-value', element.model.value);

                // building the template of the itag:
                content = '<button class="pure-button pure-button-bordered"><div class="pointer"></div><div class="btntext"></div></button>';
                // first: outerdiv which will be relative positioned
                // next: innerdiv which will be absolute positioned
                // also: hide the container by default --> updateUI could make it shown
                content += '<div class="itsa-hidden">' +
                             '<div>'+
                               '<ul fm-manage="li" fm-keyup="38" fm-keydown="40" fm-noloop="true"></ul>';
                             '</div>'+
                           '</div>';
                // set the content:
                element.setHTML(content);
            },

            /*
             * Internal hash containing all DOM-events that are listened for (at `document`).
             *
             * DOMEvents = {
             *     'click': callbackFn,
             *     'mousemove': callbackFn,
             *     'keypress': callbackFn
             * }
             *
             * @property DOMEvents
             * @default {}
             * @type Object
             * @private
             * @since 0.0.1
            */
            attrs: {
                expanded: 'boolean',
                'primary-button': 'boolean',
                value: 'string',
                'invalid-value': 'string'
            },

           /**
            * Redefines the childNodes of both the vnode as well as its related dom-node. The new
            * definition replaces any previous nodes. (without touching unmodified nodes).
            *
            * Syncs the new vnode's childNodes with the dom.
            *
            * @method _setChildNodes
            * @param newVChildNodes {Array} array with vnodes which represent the new childNodes
            * @private
            * @chainable
            * @since 0.0.1
            */
            sync: function() {
                // inside sync, YOU CANNOT change attributes which are part of `attrs` !!!
                // those actions will be ignored.

                // BE CAREFUL to start async actions here:
                // be aware that before ending, this method can run again
                // if you do, then make sure to handle possible running
                // async actions well !!!

                var element = this,
                    model = element.model,
                    items = model.items,
                    buttonTexts = model.buttonTexts,
                    value = model.value,
                    item, content, buttonText, len, i, markValue,
                    button, container, itemsContainer, renderedBefore, hiddenTimer;

                len = items.length;
                (value>len) && (value=0);
                markValue = value - 1;

                if (value>0) {
                    buttonText = buttonTexts[markValue] || items[markValue];
                }
                else {
                    buttonText = model['invalid-value'] || DEFAULT_INVALID_VALUE;
                }

                // rebuild the button:
                button = element.getElement('button');
                button.toggleClass('pure-button-primary', model['primary-button']);
                button.getElement('div.btntext').setHTML(buttonText);

                // show or hide the content, note that when not rendered before, you should use transitions
                renderedBefore = element.hasClass(CLASS_ITAG_RENDERED);
                container = element.getElement('>div');

                if (model.expanded) {
                    hiddenTimer = container.getData('_hiddenTimer');
                    hiddenTimer && hiddenTimer.cancel();
                    container.setClass(SHOW);
                    container.removeClass(HIDDEN);
                }
                else {
                    container.removeClass(SHOW);
                    // hide the layer completely: we need to access anything underneath:
                    hiddenTimer = laterSilent(function() {
                        container.setClass(HIDDEN);
                    }, 110);
                    container.setData('_hiddenTimer', hiddenTimer);
                }

                itemsContainer = element.getElement('ul[fm-manage]');
                content = '';
                for (i=0; i<len; i++) {
                    item = items[i];
                    content += '<li'+((i===markValue) ? ' class="selected"' : '')+'>'+item+'</li>';
                }

                // set the items:
                itemsContainer.setHTML(content);
            }
        });

        Itag.setItagDirectEventResponse(['blur', 'keypress']);
    }

    return window.ITAGS[itagName];
};
```