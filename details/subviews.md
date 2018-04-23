# <span id="contents">Views and SubViews</span>

- Views
    - [Simple Views](#simple)
    - [Object "Factory Pattern"](#factory)
    - [Class Views](#class_views)
- SubView Including
    - [View Inclusion](#view_subview)
    - [App Inclusion](#app_subview)

A view file contains a complete functionality of a particular part of the UI: a form, a datatable with the related toolbar, a navigation menu, etc. Views can be defined in three ways.

## [<span id="simple">1. Simple Views &uarr;</span>](#contents)

Views can be created as pure objects.

```js
/* views/list.js */
export default {
	view:"list"
}
```

or 

```js
const list = {
    view:"list"
};
export default list;
```

### Advantage 

- This is a simple way to create a view.

### Disadvantages
 
- Simple views are static and are included as they are.
- Simple views have no **init()** and other methods that classes have.

## [<span id="factory">2. Object "Factory Pattern" &uarr;</span>](#contents)

View objects can also be returned by a factory function.

```js
/* views/details.js */
export default () => {
	var data = [];
	for (var i=0; i<10; i++) data.push({ value:i });

	return {
		view:"list", options:data
	}
}
```

### Advantages

- Such views are still simple.
- Such views are dynamic.

### Disadvantages

- Such views have no **init()** or other methods that classes have.

## [<span id="class_views">3. Class Views &uarr;</span>](#contents)

Views can be defined as ES6 classes that inherit from the _JetView_ class.

```js
// views/top.js
import {JetView} from "webix-jet";
export default class TopView extends JetView {
     config(){
        return { cols:[
            { view:"menu" }
            { template:"Something here" }
        ]};
    }
}
```

- [<span id="class_contents">Advantages of Classes</span>](#class_advantages)
- [JetView Lifetime Methods](#methods)
- [Local Methods and Properties](#class_methods_properties)

### [<span id="class_advantages">Advantages of Classes &uarr;</span>](#class_contents)

- Views defined as classes are **dynamic** and each new instance can be changed when it's created.

- View classes have **init()** and other **lifetime methods** that can be redefined by users. 

- You can also define **custom methods** and **local variables**.

- All instances have their individual **inner states**. E.g. if you use the same Toolbar class to add identical toolbars at the top and at the bottom, there are two instances of the Toolbar class and the toolbars will behave independently.

- Classes have the **this** pointer that references the view inside methods and handlers.

- You can **extend** class views. Views can inherit not only from the JetView class, but from each other.

Thanks to ES6 classes, inheritance is closer to classic OOP. Inheritance can help you reuse old components for creating slightly different ones. In the case when views share many common traits, you can create a base view and create a necessary number of subclasses, in which you can redefine necessary parts of UI/logic. For example, you already have a toolbar and want to create a similar one, but with additional buttons. You can define a new _BiggerToolbar_ class view on the base of Toolbar one and add elements in the _init_ handler.

Base class:

```js
import {JetView} from "webix-jet";
export default class Toolbar extends JetView {
    config(){
      return { view:"toolbar", height:50, elements:[ ... ]};
    }
}
```

Successor class:

```js
import Toolbar from "views/toolbar";
export default class BiggerToolbar extends Toolbar {
    init(view){
        view.addView({view:"button", value:"Save"});
    }
}
```

### [<span id="methods">JetView Methods &uarr;</span>](#class_contents)

Webix UI lifetime event handlers are implemented through **JetView** class methods. Here are the methods that you can redefine:

- [config()](#config)
- [init()](#init)
- [urlChange()](#urlchange)
- [ready()](#ready)
- [destroy()](#destroy)

#### [<span id="config">config() &uarr; </span>](#methods)

This method returns the initial UI configuration of a view. Have a look at a toolbar returned by the **config()** method of the _ToolbarView_ class:

```js
// views/toolbar.js
import {JetView} from "webix-jet";

export default class ToolbarView extends JetView{
    config(){
        return {
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", options:["details", "dash"] }
            ]
        };
    }
}
```

#### [<span id="init">init\(view, url\) &uarr;</span>](#methods)

The method is called only once for every instance of a view class when the view is rendered. It is a good place to load some common data (list of options for a _select_ in a form, for example) or to change the initial UI configuration of a view returned by **config()**.

**init()** receives two **parameters**:

1. **view** - the view UI,
2. **url** - the app URL as an array.

Each array element of **url** is an object that contains three properties:

- **page** - the name of the URL element,
- **params** - parameters that you can pass with the URL,
- **index** - the index of the URL element (beginning from 1).

Let's use **init()** to change the state of a control in a simple UI.

```js
// views/toolbar.js
import {JetView} from "webix-jet";

export default class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", options:["details", "dash"] }
            ]
        };
    }
}
```

By default, a segmented button will be always rendered with the first segment active. Let's link the control state to the URL in **init()**:

```js
// views/toolbar.js
import {JetView} from "webix-jet";

export default class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", options:["details", "dash"] }
            ]
        };
    }
    init(view, url){
        if (url.length > 1) //if there is a subview
            view.queryView({view:"segmented"}).setValue(url[1].page);
    }
}
```

The segmented button is referenced by the Webix method <a  href="https://docs.webix.com/api__ui.baseview_queryview.html" target="_blank" rel="noopener noreferrer">**queryView()**</a>, while **view** is the top Webix widget in the view. _url[1].page_ is the name of the current subview (_details_ or _dash_).

This is how you can load data to a Jet class view:

```js
// views/data.js
import {JetView} from "webix-jet";
import {records} from "models/records";

export default class DataView extends JetView{
    config(){
        return {
            view:"datatable", autoConfig:true
        };
    }
    init(view){
        view.parse(records);
    }
}
```

#### [<span id="urlchange">urlChange\(view,url\) &uarr;</span>](#methods)

**urlChange()** is called every time the URL is changed. It reacts to the change in the URL after **!\#**<sup><a href="#footnote1" id="origin">1</a></sup>. **urlChange()** is only called for the view that is rendered and for its parent.

Consider the following example. The initial URL is:

```
/layout/demo/details
```

If you change it to:

```
/layout/demo/preview
```

**urlChange()** will be called for **preview** and **demo**. **demo** is not reconstructed. Such approach allows preserving parts of UI not affected by navigation, which improves performance and UX of the app. It is especially important if you have some complex widget in the parent view and do not want to fully reconstruct/reload it on subview navigation.

The **urlChange** method receives the same two **parameters** as **init**:

- **view** - the Webix widget inside the Jet view class
- **url** - the URL as an array of URL elements

**urlChange()** can be used to restore the state of the view according to the URL, e.g. to highlight the right controls.

Let's expand the previous example with a toolbar:

```js
// views/toolbar.js
import {JetView} from "webix-jet";

export default class ToolbarView extends JetView{
    config(){
        return { 
            view:"toolbar", elements:[
                { view:"label", label:"Demo" },
                { view:"segmented", options:["details", "dash"] }
            ]
        };
    }
}
```

Here's how you can highlight the right segment of the button if the URL is changed (with browser navigation buttons, for example):

```js
// views/toolbar.js
    urlChange(view, url){
        if (url.length > 1)
            view.queryView({view:"segmented"}).setValue(url[1].page);
    }
```

#### [<span id="ready">ready(view,url) &uarr;</span>](#methods)

**ready()** is called when the current view and all its subviews have been rendered. For instance, if the URL is changed to *a/b*, the order in which view class methods are called is the following:

```
config a
init a
	config b
	init b
	urlChange b
	ready b
urlChange a
ready a
```

**ready()** receives same two **parameters**:

- **view** - the Webix widget inside the Jet view class
- **url** - the URL as an array of URL elements

Here's how you can use **ready()**. There are two simple views, a list and a form for editing the list:

```js
// views/list.js
const list = {
    view: "list",
    select: true,
    template: "#value#",
    data: [{value:"one"},{value:"two"}]
};

// views/form.js
const form = {
	view: "form",
	elements:[
		{view: "text", name: "value", label: "Value"},
		{view: "button", value: "Save", width: 90}
	]
};
```

Let's include these views into one module and bind the list to the form:

```js
// views/listedit.js
import {JetView} from "webix-jet";

export default class ListEditView extends JetView{
    config(){
        return {
            cols:[
                { $subview:"list" },   //load "views/list"
                { $subview:"form" }    //load "views/form"
            ]
        }
    }
    ready(view){
        const form = view.queryView({view:"form"});
        const list = view.queryView({view:"list"});
        form.bind(list);
    }
}
```

In the example, the form will be bound to the list only when both the list and the form are rendered.

#### [<span id="destroy">destroy() &uarr;</span>](#methods)

**destroy()** is called only once for each class instance when the view is destroyed (closed and no longer present in the URL).

You can use **destroy()** to detach events that were attached by this view with **app.attachEvent()**. Events attached by **attachEvent()** are not destroyed automatically.

```js
// views/form.js
import {JetView} from "webix-jet";

export default class FormView extends JetView{
    init(){
        this.app.attachEvent("save:form", function(){
            this.show("aftersave");
        });
    }
    destroy(){
        this.app.detachEvent("save:form");
    }
}
```

### [<span id="class_methods_properties">Local Methods and Properties &uarr;</span>](#class_contents)

You can define class view methods and properties. **this** inside methods refers to the instance of the corresponding view class.

Consider a simple example. The class has a counter stored as a class property, which is declared in **init()**. There is also a method that increments it, when a button is clicked.

```js
import {JetView} from "webix-jet";
export default class Toolbar extends JetView {
    config(){
        return {
            view:"button", value:"Click me", 
            click:() => this.doClick("Clicked")
        };
    }
    init(){
        this._counter = 0;
    }
    doClick(message){
        this._counter++;
        webix.message(message+" "+this._counter);
    }
}
```

## &#x2753; Which Way to Define Views is Better

If you still doubt which way to choose for defining views, here's a summary.

All ways provide nearly the same result.

When you are using the **"class"** approach, you can define the UI configuration and methods for lifetime handlers.

When you are using the **"factory function"** approach, you can define a dynamic UI config without lifetime handlers.

When you are defining views as **const** _(simple view objects)_, you can define UI config only.

So if you are choosing between **classes** and **const**, it is flexibility VS brevity.

If you are not sure which one to use, use classes. A class with the **config()** method works exactly the same as the **const** declaration.

## Subview Including

### [<span id="view_subview">1. View Inclusion &uarr;</span>](#contents)

You can include views into each other. Views included into other views are called **subviews**, and they can be either static or dynamic.

**1. Static subviews** are imported and placed into views directly.

```js
import Menu from "views/menu";
export default class TopView extends JetView {
   config(){
       return {
            rows:[
                { view:"button" },
                Menu
            ]
       };
   }
}
```

Subviews can also be included statically with the **$subview** keyword that points either to a class or file name:

```js
import Menu from "views/menu";
export default class TopView extends JetView {
    config(){
        return {
            rows:[
                { view:"button" },
                { $subview:Menu },
                //or
                { $subview:"menu" }
            ]
        };
    }
}
```

**$subview** can also point to a hierarchy of views:

```js
export default class TopView extends JetView {
    config(){
        return {
            rows:[
                { view:"button" },
                { $subview:"menu/main" }
            ]
        };
    }
}
```

2\. Subviews which are resolved based on the URL segments are called **dynamic**. To create them, you need to put a placeholder into the UI with the help of **$subview:true**:

```js
// views/top.js
{ cols:[
   { view:"menu" },
   { $subview:true }
]}
```

For example, here are three views created in different ways:

- a class view

```js
// views/myview.js
import {JetView} from "webix-jet";

export default class MyView extends JetView {
    config() => { template:"MyView text" };
}
```

- a simple view object

```js
// views/details.js
export default Details = { 
    cols: [
        { template:"Details text" },
        { $subview:true } 
    ]    
}
```

- a view returned by a factory

```js
// views/form.js
export default Form = () => {
    view:"form", elements:[
        { view:"text", name:"email", required:true, label:"Email" },
        { view:"button", value:"save", click:() => this.show("details") }
    ]
}
```

Let's group them into a bigger view:

```js
// views/bigview.js
import myview from "views/myview";

export default BigView = {
    rows:[
        myview,
        { $subview:"/details/form" }
    ]
}
```

Mind that all these views could be put in any order you want and it doesn't depend whether they are classes or objects, e.g.:

```js
// views/bigview.js
import details from "views/details";

export default BigView = {
    rows:[
        details,
        { $subview:"/myview/form" }
    ]
}
```

#### View Inclusion into Popups and Windows

You can also include a view into a **popup or a window**:

```js
// views/some.js
import WindowView from "views/window";
...
init(){
    this.win1 = this.ui(WindowView);
    //this.win1.show();
}
```

where _WindowView_ is a view class like the following:

```js
// views/window.js
import {JetView} from "webix-jet";

export default class WindowView extends JetView{
    config(){
        return { view:"window", body:{} };
    }
    show(target){
        this.getRoot().show(target);
    }
}
```

For more details about popups and windows, [go to the "Popups and Windows" section](popups.md).

### [<span id="app_subview">2. App Inclusion &uarr;</span>](#contents)

App is a part of the whole application that implements some scenario and is quite independent. It can be a subview as well. By including apps into other apps, you can create high-level applications. E.g. here are two views:

```js
// views/form.js
import {JetView} from "webix-jet";

export default class FormView extends JetView {
    config() {
        return {
            view: "form",
            elements: [
                { view: "text", name: "email", required: true, label: "Email" },
                { view: "button", value: "save", click: () => this.show("details") }
            ]
        };
    }
}

// view/details.js
export default DetailsView = () => ({
    template: "Data saved"
});
```

Let's group these views into an app module:

```js
// views/app1.js
import {JetApp,EmptyRouter} from "webix-jet";

export var app1 = new JetApp({
    start: "/form",
    router: EmptyRouter //!
}); //no render!
```

Note that this app module isn't rendered. The second important thing is the choice of the router. As this is the inner level, it can't have URL of its own. That's why _EmptyRouter_ is chosen. [Go to the "Routers" section](routers.md) for details. 

Next, the app module is included into another view:

```js
// views/page.js
import {app1} from "app1";
import {toolbar} from "views/toolbar";

export default PageView = () => ({
    rows: [ toolbar, app1 ]
});
```

Finally, this view can also be put into another app:

```js
// app2.js
import {JetApp,HashRouter} from "webix-jet";

var app2 = new JetApp({
    start: "/page",
    router: HashRouter
}).render();
```

As a result, this is a two-level app.

[Check out the demo >>](https://github.com/webix-hub/jet-demos/blob/master/sources/viewapp.js).

Jet apps can also behave as Webix widgets, for details, check ["Deploying App"](../practice/deploy.md).

<!-- footnotes -->
- - -
<a href="#origin" id="footnote1">1</a>:
This is true if you use *HashRouter*. There's no hashbang with other routers, but this still works for *URL* and *Store* routers. The URL isn't stored only for *EmptyRouter*. For details, [go to the "Routers" section](routers.md).