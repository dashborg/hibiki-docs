---
title: Actions (Advanced)
weight: 2
---

Hibiki HTML actions are very flexible.  Here are some advanced techniques that allow you to mix
backend and frontend data, and execute dynamic code.

{{< toc >}}

## hibikiexpr

Most action data can be set as a literal string/object *or* as a *hibikiexpr*.
A *hibikiexpr* will be *evaluated* on the client side.  Any *data*, *event*, or
*callpath* value can be set to a *hibikiexpr*.

Here's an example of incrementing the $.value path by a dynamic amount.  We'll
set a context variable, and then use the context variable in the 2nd action in
an expression to increment $.value:

```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "@amt", "value": 10},
   {"actiontype": "setdata", "setpath": "$.value", "value": {"hibikiexpr": "$.value + @amt"}}
]}
```

Here's an example of a conditional expression that sets $.color to red if the new
value is less than 0:
```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "@amt", "value": 10},
   {"actiontype": "setdata", "setpath": "$.value", "value": {"hibikiexpr": "$.value + @amt"}}
   {"actiontype": "setdata", "setpath": "$.color", "value": {"hibikiexpr": "$.value >= 0 ? 'black' : 'red'"}}
]}
```

Here we'll update the customer name, and return the full customer object:
```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "$.customer.name", "data": "Mike"},
   {"actiontype": "setreturn", "data": {"hibikiexpr": "$.customer"}}
]}
```

## hibikicontext

The optional *hibikicontext* key along side the *hibikiactions* key is used to inject
context data while running the actions.  Any value in the object can be accessed using
the "@var" syntax.  This is useful when combined with *hibikiexpr* values.

Here's the example from above, but using *hibikicontext*.  @amt is set via *hibikicontext*:
```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "$.value", "value": {"hibikiexpr": "$.value + @amt"}}
   {"actiontype": "setdata", "setpath": "$.color", "value": {"hibikiexpr": "$.value >= 0 ? 'black' : 'red'"}}
 ],
 "hibikicontext": {"amt": 10}
}
```

Here we'll fire an event handler, using data from our frontend data model and our backend action.
Value is set from "$.value" (from the frontend), and mode is set using *hibikicontext*.
```
{"hibikiactions":[
   {"actiontype": "fireevent", "event": "custom1", "data": {"hibikiexpr": "{value: $.value, mode: @mode}"}}
 ],
 "hibikicontext": {"mode": "compact"}
}
```

We can use a dynamic index from *hibikicontext* to set an array value:
```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "$.arrdata[@index]", "data": 28}
 ],
 "hibikicontext": {"index": 8}
}
```

*hibikicontext* can also avoid data duplication:
```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "$.v1", "data": {"hibikiexpr": "@value"}},
   {"actiontype": "setdata", "setpath": "$.v2", "data": {"hibikiexpr": "@value"}},
   {"actiontype": "setdata", "setpath": "$.v3", "data": {"hibikiexpr": "@value"}}
 ],
 "hibikicontext": {"value": ... large json value ... }
}
```


## blobs

It can be easier to return blob data (like images) in the same response as data.  To return a blob,
use the *setdata* action, but instead of setting *data*, use the two special keys
*blobbase64* and *blobmimetype*.  *blobbase64* should be a properly padded Base64 encoding of the
blob object, and *blobmimetype* should be a valid mime type.

```
{"hibikiactions":[
   {"actiontype": "setdata", "setpath": "$.img", 
    "blobmimetype": "image/jpeg", 
    "blobbase64": "...base64 encoded data..."}
 ]
}
```

## hibikihandler

Used to return a handler block that will be parsed and evaluated by Hibiki HTML.  *hibikicontext* can
be specified for the returned block to allow the backend to inject values.

```
{"hibikihandler": "$.dataval = $.dataval + @inc; fire->dataupdated($.dataval);",
 "hibikicontext": {"inc": 5}
}
```
