<style>
.date{
    color:#999;
    font-size:80%;
    float:right;
    padding-right:10em;
}
</style>

# What's New

### Version 1.5 <span class="date">April 24, 2018</span>

- the ability to compile a Jet app into a standalone bundle and [use it in other apps or as is](practice/deploy.md)
- the ability to compile a Jet app as a component and [use it as a Webix widget](practice/deploy.md)
- [**this.$$()**](details/views.md#id) returns [widgets from the current Jet view only](details/referencing.md#controls)

### Version 1.4 <span class="date">April 11, 2018</span>

- JetView class instances can be created [in **config()**](practice/nested.md#subgrid)

### Version 1.3 <span class="date">January 11, 2018</span>

- [TypeScript support](practice/typescript.md)
- new API:
    - [getParam](details/views.md#getparam) and [setParam](details/views.md#setparam) methods to get/set URL parameters
    - [getUrl](details/views.md)
- [*UrlParam*](details/plugins.md#urlparam) plugin
- ability to set optional path for the [Locale plugin](details/plugins.md#locale)
- ["app:user:login"](details/inner_events.md#login) and ["app:user:logout"](details/inner_events.md#logout) events

### Version 1.2 <span class="date">December 12, 2017</span>

- [*show*](details/views.md#show) and [*getSubView*](details/views.md#getsub) APIs updated
- *this.ui* updated
    - allows to define [custom HTML parent](details/views.md#container)
    - can be used with [JetView classes](details/popups.md#class)
- ability to [remove *"!"* from the hash URL](details/routers.md#hash)
- ability to return a promise from the *config* method of a view
- HashRouter supports [*config.routes*](details/app_config.md#routes)
- views can define [string-to-string mapping](details/app_config.md#views)

### Webix Jet 1.0 <span class="date">September 26, 2017</span>

Major update after version 0.5:
- [views](basic/views.md) and [app modules](basic/app.md) as ES6 classes
- using [Webpack](practice/webpackconfig.md) as a module bundler
- ability to choose among [4 routers](details/routers.md)
- ability to use [6 plugins](details/plugins.md) for common tasks