---
title: Core Tags
weight: 5
---

# Node Tags

## &lt;h-text&gt;

Renders text content.  If class/style are not specified, it will render as a DOM text node.  If you
provide values for class or style, it will render as a &lt;span&gt; with class and style set.

{{ expr }} is equivalent to &lt;h-text bind="expr">&lt;/h-text>.

**Attributes**

<div style="margin-top: -15px;">

* **bind** (value) - the value to render
* **format** (string) - how to format the value (default is just converting value to a string).  see below.
* **class** (string) - class value
* **style** (string) - style value
</div>

**format** attribute is a sprintf like format specifier. uses the [sprintf-js](https://www.npmjs.com/package/sprintf-js) library to render output.  Also support three special values "json", "json-compact", and "json-noresolve" to format the given value as JSON.

## &lt;h-fragment&gt;

Groups a set of children.  Does not render as any tag (removed from the output).
Useful when combined with other attributes like *if* or *foreach*.

**Attributes**:
<div style="margin-top: -15px">

* **[children]** - the children to render
</div>

## &lt;h-dyn&gt;

Dynamically parses and renders Hibiki HTML content.  Renders the same as &lt;h-fragment&gt;.  On a parse error it will not render any output and report an error.

**Attributes**

<div style="margin-top: -15px">

* **bind** (string) - the HTML text to parse/render.
</div>

## &lt;h-children&gt;

Renders children passed to components.  Normally passed ```@children``` or a filtered set of children nodes.
The children are rendered in their *original parent's* data context.  So children do not have access to ```$c```
or any component context variables.

In order to pass data from the component to the child you must *inject* attributes (similar to how
higher order components work in React, but much easier to implement in Hibiki HTML).

**Attributes**

<div style="margin-top: -15px">

* **text** (string) - if *text* is not null, will not render any children and render the given text instead.
* **bind** (children) - the children to render.  If *bind* attribute is not passed, if it evaluates to *null* or if there are zero children to render, it will instead render its *own* children.
* **[children]** - used when *bind* is not set, is null, or contains zero children
* **inject:\*** - attributes to inject into the rendered children (see injection below).
* **datacontext** - additional context (@vars) to set when rendering children.  acts as if an extra &lt;define-vars&gt; was run right before rendering the child.
</div>

When the *bind* attribute is not provided, is null, or there are zero children to render, &lt;h-children&gt; renders its *own* children.  This is useful when writing a component that has a default UI, and can be optionally overwritten by content passed to your component (e.g. in a special "slot" child).  This is also useful if you want to *inject* attributes into a group of child nodes.

**Injection** - injected attributes overwrite any value that is already present in the child (they are not merged).  If the injected attribute is an expression (starts with ```*```) then it will have access to a special ```@node``` context variable representing the child node that is being injected.  Using the ```@node.attrs``` you can inject dynamic attributes into the node.
```@node``` contains the fields:

* **tag** - the node's tag name
* **rawtag** - raw tag name (useful if the *component* attribute or "html-" prefix was used to transform the tag)
* **uuid**
* **attrs** - the resolved attributes of the node (not including injected attributes)
* **children** - 'children' variable for the child node

Note &lt;h-children&gt; does *not* support the *foreach* attribute.

```
# most common useage, just render all children as-is
<h-children bind="@children"></h-children>

# if $args.label is set, it will render that value as text.
# if a children with 'slot' equal to 'label' are passed, it will render those children
# otherwise it will evaluate to the text '[No Label]'
<h-children text="$args.label" bind="@children.byslot['label']">[No Label]</h-children>

# will render the <input> tag with value set to 'mike' and with blue text style
<h-children inject:value="mike" inject:style="color: blue;">
   <input type="text">
</h-children>

# renders the first <input> child.  injects a change handler
<h-children inject:change.handler="$c.value = @value;" bind="@children.bytag['input'].first"></h-children>

# advanced usage.  select 'column' elements from the library named 'mylib'.
# in the first call, inject the attribute "header" and render them
# in the second call, render the children for each row and inject the 'row' object
# note that we're using <html-tr> because in HTML, standard <tr> tags cannot accept non-<td> children.
# could also have written it as <tr><td component="h-children" ...></td></tr>
<html-tr>
  <h-children bind="@children.bycomp['mylib:column']" inject:mode="header"></h-children>
</html-tr>
<html-tr foreach="(@row, @idx) in $args.rows">
  <h-children bind="@children.bycomp['mylib:column']" 
              inject:mode="row" inject:row="*raw(@row)" inject:rownum="*@idx"></h-children>
</html-tr>
```

## &lt;h-watcher&gt;

Allows you to watch a value or expression and fire a handler any time the value changes.  Care should be taken to not create infinite loops by modifiying the value you are watching in the update handler.

Normally Hibiki HTML nodes are reactive to their data already, but &lt;h-watcher&gt; allows you to bridge that reactivity into non-Hibiki code or AJAX requests.

**Attributes**
<div style="margin-top: -15px">

* **bind** - The value to watch.  Can be a simple path like ```$.data.name``` or a complex value (to watch multiple paths): ```[$.data.name, $.data.id]```.
* **fireoninit** - if true, will fire when the watcher is first initialized, otherwise the initial value does not fire the *change* event.
</div>

**Events**
<div style="margin-top: -15px">

* **change** - Fired whenever the watched value changes.  @value will be set to the watched value.
</div>

# Definition Tags

## &lt;script&gt;

Script tags work in a special way in Hibiki HTML.  When "rendered" their *src* or text content is compared against a cache of already rendered script tags.  If the value is found, nothing happens, otherwise the script src or content is run.

**Attributes**
<div style="margin-top: -15px">

* **src** - script src (also can specify text content as a child)
* **async** - run the script tag synchronously or asynchronously.  If not async, loading the script will block the library load or your app's "init" event.  Useful when you need a library loaded to use your app or library (e.g. d3).
* **[textbody]** - script text (when src is not specified)
</div>

Often it is easier (when not in a library) to load your scripts outside of the Hibiki tag which will naturally block your app until the script is loaded.

## &lt;define-vars&gt;

Creates a new set of context variables (@vars) that can be used by the siblings following the &lt;define-vars&gt; and their descendents.  A define-vars expression is just a sequence of simple assignments (it does not support any other type of statement).  When you re-define a variable that already exists, it *masks* the old one in the current scope.  The old value will be available when this define-vars goes out of scope.

**Attributes**
<div style="margin-top: -15px">

* **datacontext** - the variables to assign (as an attribute)
* **[textbody]** - the variables to assign (as body text)
</div>

**Allowed Define Vars:**
```
# can assign using @ or without
# can access global scope
# can separate assignments with "," or with ";" (or mixed)
# new variables can reference variables just created (using @) or previously defined @vars
<define-vars>
    x=5, y = @x*2;
    @z = {name: "mike", id: $.global_id}
</define-vars>

# can use attribute 'datacontext'
# can access $args and $c
<define-vars datacontext="value = raw($args.value) ?? ref($c.value)"></define-vars>
```

**Invalid Define Vars:**
```
<define-vars>
  $.x = 5;        # cannot set variables in $ or $state scopes
  @foo.x = 10;    # cannot assign to sub-indexed variables
  if ($.x) {      # invalid statement, can only do simple assignments
      @bar = 10;
  }
  log(@foo);                     # non-assignment statement
  @x = //@local/assign-value();  # cannot call handlers
</define-vars>
```

## &lt;define-handler&gt;

Defines a handler.  The text body of the tag is parsed as the handler definition.

**Attributes**:
<div style="margin-top: -15px">

* **name** - name of handler (either *//@local/[name]* or *//@event/[name]*).
* **[textbody]** - the Hibiki HTML code that defines the handler
</div>

## &lt;define-component&gt;

Defines a component.  The children tags are parsed as the component definition.

**Attributes**:
<div style="margin-top: -15px">

* **name** - name of handler (either *//@local/[name]* or *//@event/[name]*).
* **componentdata** - datacontext to initialize the $c root
* **[children]** - component definition
</div>


## &lt;import-library&gt;

Contains components, handlers, and other definitions for a library.

**Attributes**:
<div style="margin-top: -15px">

* **name** - name of library
* **[children]** - components, handlers, &lt;script&gt;, &lt;link&gt; (css)
</div>
