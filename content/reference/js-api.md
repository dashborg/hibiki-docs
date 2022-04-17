---
title: JS API
weight: 15
---

{{<toc>}}

Hibiki is designed so that for simple projects you should not need to ever write a line of JavaScript.  However, there *are*
times when you may need to interact with existing JS code, or an existing JS library where there is a need to bridge
data and functionality between Hibiki and the browser's JS layer (especially when writing your own Hibiki libraries).

The Hibiki JS API allows you to interact with your running Hibiki code by:

* Getting data from the Hibiki data model
* Setting data into the Hibiki data model (potentially causing the UI to update)
* Watching for Hibiki data model changes (getting notified via a callback)
* Registering functions that can be called as Hibiki handlers
* Registering functions that can be called as Hibiki expressions

## HibikiGlobalConfig

To update Hibiki settings before the Hibiki applications are rendered you can define a
global JS object called `HibikiGlobalConfig`. It can
be used to set global options that affect the rendering of the Hibiki tags, as well as 
firing callbacks when the Hibiki applications are initialized.  It provides two hooks related to
application lifecycle `preRenderfHook` and `postRenderHook`.  The `preRenderHook` is called right after
the application is initialized but before the initial UI rendering.  The `postRenderHook` is
called after the first successful UI render.  Both hooks are called with 2 elements, the `HibikiState`
object, and the HTMLElement that the UI will be rendered into.

## HibikiState

Every `<template hibiki>` element is controlled by its own `HibikiState` object.  The `HibikiState` object 
contains the controller for the UI, and the data that is used to render the UI.  You use the `HibikiState`
object to interact with your Hibiki application.

Hibiki applications are initialized after the `DOMContentLoaded` event is fired (unless you manually
create and initialize your applications).  So any interaction with the `HibikiState` objects should also wait
until after they are initialized.

The easiest way to get a `HibikiState` object is via the global variable `window.HibikiState`.
This works great for most cases when there is only one Hibiki application defined on a page (it will
always be set to the *last* Hibiki application that was initialized on a page).

When there are multiple application on one page, you can add a `name` attribute to any of your Hibiki
tags, e.g. `<template hibiki name="test1"> and <template hibiki name="test2">`.  Now to access
those states you can use the `window.Hibiki.States` global variable (which is an object that
maps names to `HibikiState` objects).  So `test1`'s state can be accessed as
`window.Hibiki.States["test1"]`.  As `States` is just a plain JS object, it can also be looped
over to find all of your named states.  Note that Hibiki tags without a `name` attribute will
not be added to `States`.

## Getting / Setting Data

Once you have a `HibikiState` object you can get data from the data-model using
`state.getData(path)` and set data using `state.setData(path, data)`.

This example shows how to test and set `$.color` in the Hibiki data model from
JavaScript before the application is rendered.
```
// hooks get called with (state, htmlelem)
function setDefaultColorHook(state) {
    let currentColor = state.getData("$.color");
    if (currentColor == null) {
        state.setData("$.color", "green");
    }
}

window.HibikiGlobalConfig = {
    preRenderHook: setDefaultColorHook,
};
```

This example shows how you could update the Hibiki data model from a click handler
attached to a external HTML element.  Note that you should never manually attach
an event listener to an element that is controlled by Hibiki, only
elements that are outside of the Hibiki application:
```
function externalUpdateColor() {
    let state = window.HibikiState;   // gets the HibikiState object
    state.setData("$.color", "red");
}

document.getElementById("mybutton").addEventListener("click", externalUpdateColor);
```

In the console (to inspect/set data):
```
// dumps the entire global Hibiki data model
HibikiState.getData("$");

// dump the value stored in $.color to the console
HibikiState.getData("$.color");

// update the color
HibikiState.setData("$.color", "purple");
```


## Registering JS Handlers

Registering JS handlers allows you to call some JavaScript code directly from
your Hibiki application as a "handler".  Note that local handlers can
be registered at *any time after* the Hibiki `<script>` tag has been included.
Handlers get passed one argument, a `HibikiRequest` object that contains
the data passed to the handler as well as other metadata.

Just like normal Hibiki handlers, the handler can be *pure* and only return
data, or it can contain *actions* which operate on the data-model directly.

Here's an example:
```
// this handler operates on the data model directly by calling req.setData
window.Hibiki.registerLocalJSHandler("/set-color-red", (req) => {
    req.setData("$.color", "red");
});

window.Hibiki.registerLocalJSHandler("/get-data", (req) => {
    // can run any JS here. the return value will be
    // available to the Hibiki code as the return value of the handler.
    // req.data is set as an object containing the data parameters passed to the handler.
    return {x: req.data.val + 5, y: 10};
});

window.Hibiki.registerLocalJSHandler("/get-promise-data", (req) => {
    // if you return a Promise, it will be awaited and resolved
    return fetch('/some/api').then((resp) => resp.json());
});
```

Once you've registered a handler it can be called from inside of your
Hibiki code by using `//@local/[handler-path]` just as if it was
defined using a `<define-handler>` tag:
```
<!-- this handler calls req.setData directly, so it updates the data-model -->
<a click.handler="//@local/set-color-red()">Set Red</a>

<!-- here we're passing x=10 into get-data.  It will return {x: 15, y:10} -->
<a click.handler="$.mydata = //@local/get-data(x=10)">Get Data</a>

<!-- will resolve the promise returned and get set $.remote_data to the json() data of the fetch response -->
<a click.handler="$.remote_data = //@local/get-promise-data()">Get Promise Data</a>
```


