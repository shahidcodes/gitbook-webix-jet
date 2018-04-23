# <span id="contents">Using Jet Views as Webix Widgets</span>

- [Jet views in Webix Multiview/Tabview](#multiview)
- [Jet views in Webix Dashboard](#dashboard)
- [Jet views as subgrids of Webix Datatable](#subgrid)
- [Using _addView()_ with Jet views](#add_view)

## [<span id="multiview">Placing Jet views into Multiview / Tabview &uarr;</span>](#contents)

You can place Jet views inside Multiview.

![](../images/multiview.png)

#### 1. Multiview + Segmented control

Let's create a Jet view that will include Multiview with other Jet views. First, create Multiview and put Jet views as _$subviews_ into each cell:

```js
// views/top.js
import {JetView} from "webix-jet";

export default class TopView extends JetView {
	 config(){
		return {
			rows: [
				{ view:"segmented", multiview:true, options:[
					"Dashboard", "Meta Info", "Settings"
				]},
				{ view:"multiview", cells:[
					{ $subview:"childview1" },	//load views/childview1.js
					{ $subview:"childview2" }	//load views/childview2.js
				]}
			]
		};
	}
}
```

Where *childview1* and *childview2* are Jet views like this:

```js
// views/childview1.js
import {JetView} from "webix-jet";

export default class ChildView1 extends JetView {
	config(){
		return {
			template:"Admin view 1 <br> Dashboard"
		};
	}
};
```

Next, let's add a control to switch between Multiview cells - a segmented button. *segmented* needs IDs of the subviews. It is very important not to define IDs outside *top.js*, because it is unsafe. A better approach is to give IDs to the contents of the Multiview cells: 

```js
// views/top.js
import {JetView} from "webix-jet";

export default class TopView extends JetView {
	 config(){
		return {
			rows: [
				{ view:"segmented", multiview:true, options:[
					"Dashboard", "Meta Info"
				]},
				{ view:"multiview", cells:[
					{ $subview:"childview1", id:"Dashboard" },	//load views/childview1.js
					{ $subview:"childview2", id:"Meta Info" }	//load views/childview2.js
				]}
			]
		};
	}
}
```

[Check out the solution on GitHub >>](https://github.com/webix-hub/jet-demos/blob/master/sources/tabbar.js)

#### 2. Tabview

![](../images/tabview.png)

You can also put subviews into a Tabview. In this case you do not need IDs, you need to import the subviews and place them into the *body* of Tabview cells:

```js
// views/top.js
import {JetView} from "webix-jet";
import ChildView1 from "views/childview1";
import ChildView2 from "views/childview2";

export default class TopView extends JetView {
	 config(){
		return {
			type: "space",
			rows: [
				{ view:"tabview", cells:[
					{ header:"Dashboard", body: ChildView1 },
					{ header:"Meta Info", body: ChildView2 }
				]}
			]
		};
	}
}
```

[Check out the solution on GitHub >>](https://github.com/webix-hub/jet-demos/blob/master/sources/tabbar.js)

## [<span id="dashboard">Jet Views in Webix Dashboard &uarr;</span>](#contents)

You can put Jet views on panels of [Webix Dashboard](https://webix.com/widget/dashboard/).

![webix jet with webix dashboard]()

#### Static panels

Create the views, e.g.:

```js
// views/admin1.js
import {JetView} from "webix-jet";
export default class AdminView1 extends JetView {
	config(){
		return {
			template:"Admin view 1 <br> Dashboard"
		};
	}
};
```

Create a view with Dashboard:

```js
// views/dashboard.js
import {JetView} from "webix-jet";
export default class DashboardView extends JetView {
	config(){
		return {
            view:"dashboard",
            gridColumns:4, gridRows:4,
            cellHeight: 200
		};
	}
}
```

After that, you can create a panel in _cells_ and place AdminView1 into the panel body:

```js
// views/dashboard.js
import {JetView} from "webix-jet";
import AdminView1 from "views/admin1.js";
export default class TopView extends JetView {
	config(){
		return {
			view:"dashboard",
            gridColumns:4, gridRows:4,
            cellHeight: 200,
            cells:[
                {
                    view:"panel",
                    x:0, y:0, dx:1, dy:1,
                    body:AdminView1
                }
            ]
		};
	}
}
```

#### Dynamic panels

You can also create a [dynamic dashboard](https://blog.webix.com/webix-dashboard-layout-how-to-build-appealing-dynamic-dashboards/) where users will add panels themselves. For example, panels can be dragged from a list like this:

```js
// views/dashboard.js
import {JetView} from "webix-jet";
import AdminView1 from "views/admin1.js";
import AdminView2 from "views/admin2.js";
...
export default class TopView extends JetView {
	config(){
		return {
			cols: [
				{ view:"list", id:"list",
                    width:200,
                    drag:"source",
                    template:"#value# - (#dx#x#dy#)",
                    data:[
                        { id:"1", value:"AdminView1", dx:1, dy:1 },
                        { id:"2", value:"AdminView2", dx:1, dy:2 },
                        ...
                    ]
				},
				{
					view:"dashboard", id:"grid",
                    gridColumns:4, gridRows:4,
                    cellHeight: 200
				}
			]
		};
	}
}
```

Dynamic Dashboard panels can be created with a [_factory_](https://docs.webix.com/api__link__ui.dashboard_factory_config.html) instead of _cells_.

Import all the Jet views you want to place on panels in _DashboardView_ and save them in class properties:

```js
// views/dashboard.js
import {JetView} from "webix-jet";
import AdminView1 from "views/admin1.js";
import AdminView2 from "views/admin2.js";
...
export default class DashboardView extends JetView {
	config(){
		return {
            cols: [
				// ...list
				{
					view:"dashboard", id:"grid",
                    gridColumns:4, gridRows:4,
                    cellHeight: 200
				}
			]
		};
    }
    init(){
        this.AdminView1 = AdminView1;
        this.AdminView2 = AdminView2;
        ...
    }
}
```

Now let's define the factory that will place Jet views on panels:

```js
// views/dashboard.js
import {JetView} from "webix-jet";
import AdminView1 from "views/admin1.js";
import AdminView2 from "views/admin2.js";
...
export default class DashboardView extends JetView {
	config(){
		return {
            cols: [
				// ...list
				{
					view:"dashboard", id:"grid",
                    gridColumns:4, gridRows:4,
                    cellHeight: 200,
                    factory:(obj) => {
                        obj.view = "panel";
                        obj.resize = true;
                        obj.body = this[$$("list").getItem(obj.name).value];
                        return obj;
                    }
				}
			]
		};
    }
    init(){
        this.AdminView1 = AdminView1;
        this.AdminView2 = AdminView2;
        ...
    }
}
```

[Check out the demo >>](https://github.com/webix-hub/jet-demos/tree/master/sources/dashboard.js)

## [<span id="subgrid">Jet Views as Subgrids of Webix Datatable &uarr;</span>](#contents)

You can also nest Jet views of any complexity into Datatable by including them into _Datatable subviews_ <sup><a href="#footnote1" id="origin1">1</a></sup>.

![Webix Jet view as a subgrid]()

Let's add subviews into this datatable:

```js
// views/top.js
import {JetView} from "webix-jet";
export default class TopView extends JetView {
	 config(){
		return {
			view:"datatable",
			columns:[
				{ id:"title",   header:"Title", sort:"string",
				template:"{common.subrow()} #title#", width:220 },
				{ id:"year",    header:"Year",  width:100, sort:"int"},
        		{ id:"votes",   header:"Votes", width:100,  sort:"int"}
			],
			data:[
				{ id:1, title:"The Shawshank Redemption", year:1994, votes:678790 },
				{ id:2, title:"The Godfather", year:1972, votes:511495 },
				{ id:3, title:"The Godfather: Part II", year:1974, votes:319352 }
			]
		};
	}
}
```

One Jet view class will be used for all subviews, but with different data. Let's use the ability to create Jet class instances with constructors.

<span style="color:red;border:1px solid pink;border-left:5px solid red;border-radius:2px;display:block;padding:10px;"><b>Note</b>: This will work only for synchronous Jet views.</span>

Let's define the constructor that will call the constructor of the JetView class and then get the right data from a parent row of Datatable:

```js
// views/subgrid.js
import {JetView} from "webix-jet";
export default class SubLayout extends JetView {
	constructor(app, name, data){
		super(app, name);
		this.customData = data;
	}
	config(){
		return {
			rows:[
				{ type:"header", template: () => this.customData.title },
				{ template:"Subview", height: 30 }
			]
		}
	}
}
```

Let's include _SubLayout_ into the Datatable in _TopView_. **subview** of Datatable can be defined as a function with two parameters:
- **obj** - and object with the item data,
- **target** - the HTML node below the row where the subview will be opened.

The _subview_ function must return the UI of the subview.

You can create instances of _SubLayout_ with right data inside **config()** of _TopView_. Call the constructor and pass the data as the third parameter:

```js
// views/top.js
import {JetView} from "webix-jet";
export default class TopView extends JetView {
	config(){
		return {
			view:"datatable",
			subview: (obj, target) => {
				var sub = new SubLayout(this.app, "", {
					title: obj.title 
				});
				//...
			},
			//datatable config
		};
	}
}
```

Use **this.ui()** to place the _SubLayout_ below the row of Datatable and the return the UI of the newly created Jet view:

```js
// views/top.js
import {JetView} from "webix-jet";
export default class TopView extends JetView {
	config(){
		return {
			view:"datatable",
			subview: (obj, target) => {
				var sub = new SubLayout(this.app, "", {
					title: obj.title 
				});
				this.ui(sub, { container: target });
				return sub.getRoot();
			},
			//datatable config
		};
	}
}
```

## [<span id="add_view">Using _addView()_ with Jet Views &uarr;</span>](#contents)

You can use Webix **addView()** method to add Jet views into Webix layouts <sup><a href="#footnote2" id="origin2">2</a></sup>:

```js
this.$$("layout").addView(WebixJetView);
```

So you can add new elements on the fly, which is especially important for Dashboard. Also, **addView()** can be used with Layout, Tabview, Multiview, Carousel, etc.

[Check out the demo >>](https://github.com/webix-hub/jet-demos/tree/master/sources/addview.js)

- - -
<a id="footnote1" href="#origin1">1 &uarr;</a>, <a id="footnote2" href="#origin1">2 &uarr;</a>:
Starting with Webix 5.3