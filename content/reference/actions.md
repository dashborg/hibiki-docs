---
title: Actions
weight: 1
---

Hibiki HTML handlers run *actions* to manipulate the page state.  Actions can be run by
frontend Hibiki HTML code, or they can be returned as JSON from backend code.

To return actions as JSON return an object with a key named "hibikiactions":
```
{
    "hibikiactions": [action-json, action-json, ...],
}
```

### setdata

The *setdata* action updates data in the data model:
* *actiontype* - "setdata"
* *setpath* - the path to set
* *data* - literal value to set
* *setop* (optional) - the *setop* to run, defaults to "set"

Here are some example *setdata* actions:
```
{"actiontype": "setdata", "setpath": "$.color", "data": "blue"}
{"actiontype": "setdata", "setpath": "$.customers[5].first_name", "data": "mike"}
{"actiontype": "setdata", "setpath": "$.points", "data": [[0,5], [2,8], [-1, 4]]}
{"actiontype": "setdata", "setpath": "$.points", "setop": "append", "data": [8, 12]}
{"actiontype": "setdata", "setpath": "$c.points", "data": 80}
```

### setreturn

The *setreturn* action sets the handler's return value.  It is up to the frontend code to
assign or discard the return value.  Note that if you return a JSON body that does not have
the "hibikiactions" key, it gets implicitly turned into a "setreturn" value.

* *actiontype* - "setreturn"
* *data* - literal value to return

```
{"actiontype": "setreturn", "data": 8}
{"actiontype": "setreturn", "data": {"first_name": "Mike", company: "Dashborg", "status": "active"}}
```

### callhandler

The *callhandler* action calls another handler.  Use *setpath* to capture the return value.

* *actiontype* - "callhandler"
* *callpath* - the handler path (string)
* *data* - data to pass to the handler, must be null or an object (cannot be array or scalar value)
* *pure* (optional) - set to true to run as a pure handler (no side effects, only provides a return value)

```
{"actiontype": "callhandler", "callpath": "/handler2", "data": {"x": 5}}
{"actiontype": "callhandler", "callpath": "/@local/render-d3", "data": {"points": [5, 8, 22, 58]}, "pure": true}
{"actiontype": "callhandler", "callpath": "/@app/get-image", "setpath": "$.img"}
```

### fireevent

The *fireevent* action will fire or bubble an event at the point where this handler is running.

* *actiontype* - "fireevent"
* *event* - the event name (string)
* *bubble* (optional) - set to true if this event should bubble
* *data* - data associated to the event (becomes context vars, e.g. @val, or @data)

```
{"actiontype": "fireevent", "event": "custom1", "data": {"value": 22}}
{"actiontype": "fireevent", "event": "click", "bubble": true}
{"actiontype": "fireevent", "event": "submit", "data": {"first_name": "Mike", value: 42}}
```

### throw

The *throw* action will throw an error (the rest of your actions will not be run).  Similar
to firing an "error" event, with bubble set.

* *actiontype* - "throw"
* *data* - error message (string)

```
{"actiontype": "throw", "data": "Validation Error, no Customer ID set"}
{"actiontype": "throw", "data": "Internal Error"}
{"actiontype": "throw", "data": "DB Error, cannot connect to database"}
```

### ifblock

The *ifblock* action will create a dynamic if/then/else statement.  First data
is evaluated.  If true, the actions["then"] will be run.  If false
actions["else"] will be run.  Either actions block can be empty.

Normally for backend code, this action only makes sense when combined with
*hibikiexpr* (see [Actions - Advanced](/reference/actions-advanced/))


```
{"actiontype": "ifblock", "data": true,
 "actions": {"then": [... actions ...], "else": [... actions ...]}}

{"actiontype": "ifblock", "data": {"hibikiexpr": "$.loadcount > 5"},
 "actions": {"then": [... actions ...]}}
```

### html

The *html* action will overwrite the current Hibiki HTML, and replace it with the given HTML
(used to dynamically change pages).  Note that configuration data, and frontend data model will
be retained.

* *actiontype* - "html"
* *html* - the HTML to render

```
{"actiontype": "html", "html": "<h1>header</h1>..."}
{"actiontype": "html", "html": "<p>some text to render</p>"}
```

### log

The *log* action is for debugging.  It will log a value to the console, with optional
debugging stack information.  It can also fire a browser alert.

* *actiontype* - "log"
* *data* - the data to log.
* *debug* (optional) - if set to true, will log additional debug stack information
* *alert* (optional) - if set to true, it will fire a browser alert

```
{"actiontype": "log", "debug": true, "data": ["The Value of X: ", 85]}
{"actiontype": "log", "alert": true, "data": "Not Logged In"}
{"actiontype": "log", "data": "WARNING: x was null"}
```

### nop

The *nop* action does not do anything.  Used as a placeholder.

* *actiontype* - "nop"

```
{"actiontype": "nop"}
```
