---
title: Input Controls
---

{{< toc >}}

## Standard Managed Elements

* &lt;input type="text"&gt;
* &lt;input type="password"&gt;
* &lt;textarea&gt;
* &lt;select&gt;
* HTML5 &lt;input&gt; types: color, date, datetime-local, email, month, number, search, tel, time, url, week, and datetime

For **2-way** data-binding, set ```value.bindpath```.  The element's value will be read from the provided path, and on change the bindpath will be updated.  A ```change``` event will fire before the bindpath is updated with ```@value``` set to the new value.

For **1-way** data-binding, add the ```bound``` attribute and set ```value``` appropriately.  The element's value will be read from the provided path.  A ```change``` event will fire with ```@value``` set to the new value, it is up to your code to update the value if appropriate.

If neither ```value.bindpath``` nor ```bound``` is set, the component will not be managed by Hibiki HTML.

Note that for &lt;select&gt; if the ```multiple``` attribute is set, the bindpath and event ```@value``` will be an array.  When using a managed &lt;select&gt; the ```selected``` attribute on individual &lt;option&gt; tags is ignored.

## &lt;input type="radio"&gt;

For **2-way** data-binding set ```formvalue.bindpath``` (do not set the ```checked``` attribute).  When this radio button is selected, the bindpath will be set to this radio button's ```value```.  The ```checked``` attribute will be set automatically whenever the bindpath matches this radio button's value.  A ```change``` event will fire whenever this radio button is selected (before the the bindpath is updated) with ```@value``` always set to ```true```.  *Radio groups* can be managed by setting each button's ```formvalue.bindpath``` to the same value.

For **1-way** data-binding add the ```bound``` attribute and set ```value``` and ```checked``` appropriately.
Radio inputs cannot be deselected by the user, so the ```change``` event only fires when a radio button becomes
selected (```@value``` will always be ```true```).  Your code must manage the ```checked``` attributes appropriately
for other radio controls in the same group.

If neither ```formvalue.bindpath``` nor ```bound``` is set, the component will not be managed by Hibiki HTML.

## &lt;input type="checkbox"&gt;

For simple boolean **2-way** control set ```checked.bindpath```.  The element's checked state will be read from the bindpath as a boolean, and when it is checked or unchecked the bindpath will be updated to ```true``` or ```false``` as appropriate.  A ```change``` event will fire before the bindpath is updated with ```@value``` set to the *new* checkbox's state (before bindpath is updated).

To manage *checkbox groups* with **2-way** control, set ```formdata.bindpath``` (do not set the ```checked``` attribute).  The bindpath will be set to an array (set) taken from the checkbox's ```value``` attribute.  When a checkbox is checked, its value will be added to the set.  If a checkbox is unchecked its value will be removed from the set.  *Checkbox groups* can be managed by setting each checkbox's ```formvalue.bindpath``` to the same value.  A ```change``` event will be fired whenever the checkbox is checked or unchecked with ```@value``` set to ```true``` or ```false``` as appropriate.

To manage a checkbox with **1-way** control, add the ```bound``` attribute and set ```checked``` appropriately.  A ```change``` event will fire when the user checks or unchecks the control with ```@value``` set to ```true``` or ```false``` as appropriate.  Your code must manage the ```checked``` attribute appropriately.

If neither ```checked.bindpath```, ```formvalue.bindpath``` nor ```bound``` is set, the component will not be managed by Hibiki HTML.

If both ```checked.bindpath``` and ```formdata.bindpath``` are set, ```formdata.bindpath``` will be ignored.

## &lt;input type="file"&gt;

HTML file inputs do not allow their value to be set.  If ```value.bindpath``` is set, on change,
a HibikiBlob will be written to the specified path.  If no file(s) are selected the bindpath will be set
to ```null```.  A ```change``` event will fire before the bindpath is updated with ```@value``` set to the new value.

Note that if the ```multiple``` attribute is set, the bindpath and event ```@value``` will be an array of HibikiBlobs.

## &lt;input type="hidden"&gt;

As a convenience, hidden inputs can set (output only) their *value* into the data-model using
```formvalue.bindpath```.  If their *value* is dynamic, the bindpath will be set each time the
value changes.

## &lt;form&gt;

If you do not provide a ```submit.handler``` for your form element it will not be managed by Hibiki HTML (will submit with its action attribute as normal).

If you do set ```submit.handler``` the form will not submit like a normal HTML form, it will instead run the submit handler.  The submit handler will receive ```@formdata``` which is set with the values of all of the input controls associated with the form (including file inputs which will set HibikiBlob objects).  ```@formdata``` can be passed directly to call handlers as ```@data``` to pass all the data elements to the handler: ```GET /api/handler(@data=@formdata)```.

## Other &lt;input&gt; Types

Other input types including: submit, button, image, and reset are not managed.  Their attributes
work normally.  They do *not* fire ```change``` events (but will fire ```click``` events).

