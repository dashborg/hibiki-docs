---
title: Expression Reference
weight: 3
---

Hibiki HTML features a full expression language to bind your HTML to the frontend data model.
Some attributes like ```bind``` use expressions directly.  Other attributes, and style declarations
are just plain strings and are not evaluated by default.  Any attribute or style property can be
evaulated by using a ```*``` as it's first character.

Here are some examples:

```
Assuming that $.data = {x: 5, y: 20}

# bind expressions are always evaluated:
<h-text bind="$.data.x"></h-text>:        =>   5

# normal attributes and style properties are not evaluated
<div a="1" style="font-size: 14px">       =>   <div a="1" style="font-size: 14px">

# using a * will turn any attribute or style property into an expression
# like React, Dashborg will add "px" to numbers for CSS attributes where that makes sense.

<td colspan="*$.data.x">                  =>   <td colspan="5">
<div style="font-size: *$.data.y">        =>   <div style="font-size: 20px">
<div style="height: *$.data.x + '%'">     =>   <div style="height: 5%">
<div style="font-weight: *$.data.x > 2 ? 'bold' : 'normal'">  => <div style="font-weight: bold">

# inside of a foreach loop, $local is bound to each value in sequence
<h-foreach bind="[1,2,3,$.data.x]">       =>   2
  <h-text bind="$local * 2"></h-text>          4
</h-foreach>                                   6
                                               10
```

## Specifying Data Paths

All data paths start with "$" or "@".  ```$``` is used for data roots
and ```@``` is for special contextual values.

* ```$``` - Accesses data from the data root (e.g. ```$.data.rows``` or ```$.state.open```).
* ```@``` - Binds to contextual data (e.g. ```@rownum``` or ```@value```).
* ```$state``` - Binds to Hibiki HTML state data (url params, page title, other internal Hibiki HTML data).

After you specify the root, you can access array elements by using square brackets with numeric
values (```$.array[0], $.array[4], $.array[$.data.index]```) and you can access map fields using dot notation or square
brackets with string values (```$.data.map.x, $.data.map.bar, $.data.map["y"]```).

## Literals

* Strings - can be either single or double quoted.  Remember normal HTML entity encoding applies before any Dashborg processing so ```&quot;``` becomes a quote character.
  * Double Quoted - Allow escapes using ```\```.  Use ```\n``` for newlines (n, b, f, r, t, v are valid escapes).  Can also be used to escape any other character (most useful for escaping double quotes ```"```).
  * Single Quoted - No escapes allowed.
* Numbers - integers and floating point numbers, base 10 format, no exponents. (regexp: ```/[-+]?[0-9]*\.?[0-9]+/```).
* Arrays - Use square brackets, comma separated values, trailing comma is okay. (```[2, "hello", 5, ]```).
* Maps - Use braces, colon after keys, commas between key-values, trailing comma is okay.  Use quotes for keys that are not identifier safe. (```{x: 5, "abc#foo": 10}```).
* ```true```, ```false``` - for boolean values.
* ```null``` - no value
* Note that true, false, and null, are keywords (not allowed for identifiers, dot-notation, or unquoted map keys).

## Operators

These are the supported Dashborg operators, listed in increasing precedence order:

| Operator Type | Op | Notes |
|---------------|----|-------|
| **Ternary Conditional** | ? : | [conditional-expression] ? [true-value] : [false-value] |
| **NullishCoalescing**   | ??  | Like ```||``` but returns first non-null value (instead of first truthy value).  See [Nullish Coalescing Reference (Mozilla)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator)
| **Logical Or** | \|\| | Short circuits.  Returns first truthy value, or ```false``` |
| **Logical And** | && | Short circuits.  Returns first value if falsey, otherwise returns second value. |
| **Equality** | ==, != | Shallow equality tests |
| **Relational** | >=, <=, >, < ||
| **Additive** | +, - | + will work with strings for concatenation |
| **Multiplicative** | *, /, % ||
| **Unary** | !, -, + ||

## Functions

To call a function you use ```fn:[function-name]([arguments])```, 
e.g. ```fn:len($.data.expr)```.  Function names are not case sensitive.

| Function | Notes |
|----------|-------|
| fn:len     | arrays: length, objects/maps: number of keys, scalars: 1, null: 0 |
| fn:indexof | calls arg1.indexOf(arg2, [arg3]).  See [String.indexOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf) |
| fn:min | minimum of all arguments |
| fn:max | maximum of all arguments |
| fn:slice | Like the JavaScript slice function.  First argument is an array, 2nd is start element (inclusive), 3rd element is the end element (exclusive). See [Array.slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) |
| fn:splice | Like the JavaScript splice function, but does not modify the first argument, will return the newly spliced array.  See [Array.splice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) |
| fn:push | Pushes values to the end of an array.  Does not modify the input array, will return the new array. |
| fn:pop | Pops a value off the end of an array.  Does not modify the input array, will return the new array. |
| fn:setadd | First argument is an array, will conditionally the rest of the arguments to the array if they are not already in the array (checked with "==").  Returns a new array. Useful for manipulating checkbox groups, or multi-select values. |
| fn:setremove | First argument is an array, will conditionally remove the rest of the arguments from the array if they are present.  Returns a new array. |
| fn:sethas | First argument is an array, will check if the 2nd argument is inside the array (checked with "==").  Returns true/false. |
| fn:int   | converts argument to integer (truncates floats and converts strings to integer or NaN).  See [parseInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt) |
| fn:float | converts argument to floating point number (converts strings to float or NaN).  See [parseFloat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat) |
| fn:str | converts its argument to a string |
| fn:bool  | converts argument to boolean |
| fn:js    | call an external js function.  1+ arguments, first argument is a string, rest of the args get converted to a parameter list.  window[arg1].apply(null, args2-n) |
| fn:jseval  | evaluates its argument as javascript.  calls eval(arg1). |
| fn:merge | merges two objects together.  Keys from righmost arguments replace keys from leftmost arguments |
| fn:split   | splits a string into an array.  1st argument is string to split, 2nd argument is string to split on.  See [split](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) |
| fn:ts      | equivalent to javascript Date.now() (unix timestamp in milliseconds) |
| fn:now     | equivalent to javascript Date.now() (unix timestamp in milliseconds) |
| fn:substr  | *arg1*.substr(*arg2*, *arg3*).  See [substr](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/substr) |
| fn:sprintf | sprintf(*arg1*, ...) |
| fn:startswith | *arg1*.startsWith(*arg2*) |
| fn:endswith | *arg1*.endsWith(*arg2*) |
| fn:match   | creates a regex with *arg2* and and *arg3* as options.  tries to match *arg1*.  <br>e.g. *arg1*.match(new Regexp(*arg2*, *arg3*)) |
| fn:uuid | creates a UUIDv4 |
| fn:json | converts its argument to JSON |
| fn:jsonparse | parses argument from JSON to an object |
| fn:bloblen | returns length of blob |
| fn:blobmimetype | returns blob mimetype |
| fn:blobastext | returns text version of blob (only if mimetype starts with "text/") |
| fn:blobasbase64 | returns base64 encoded version of blob |
| fn:blobname | returns the file name from the blob object |

## Special Data Roots

| Data Root | Notes |
|-----------|-------|
| $ | Alias for $data (global data root) |
| @ | Alias for $context |
| $data | Returns root of frontend data model (global root) |
| $state | Returns Hibiki state data (used for interacting with internal Hibiki HTML state) |
| $context | Returns contextual data -- e.g. iteration index (@index), or input value in data (@value).  $context.value is the same as @value. |
| $contextstack | Returns an array (stack) of context data.  Used for nested loops to access outer loop's data.  $context is the same as $contextstack[0] (enclosing contexts have larger indexes).  $contextstack[0].value is the same as @value. |

## Handler Action Statements

Hibiki HTML handlers (handler, click.handler, change.handler, mount.handler, init.handler, etc.) execute actions.  The Hibiki HTML parser will convert these action statements into [Hibiki Actions](/reference/actions/).  Every action
that can be parsed can also be represented as JSON and vice-versa.

Statements must be separated with a ```;```.  Trailing ```;``` is allowed.

#### setdata action

Used to set values into the data model.  Note you can only set values into $data ($), $state, or $context (@).
Context values are local to the handler block (just used for temporary variables).

```
$state.x = 5;
$.job = {jobid: 22, jobname: "Test Job"};
@temp = 5 * $state.val;
@temp2 = @temp * @temp;
$.newval = @temp2;
@foo = /test/handler;
```

#### setreturn action

Used to return values from a handler.  Does *not* terminate the running handler, just sets the return value.

```
setreturn(5);
setreturn({x: 10, y:20});
setreturn(null);
```

#### callhandler action

Call handlers always start with ```/```.  They consist of 3 parts, an optional *module*, an optional *path*, and an optional *path fragment*.  Either module or path must be specified (path will default to ```/```).

```
/@[module-name]/[path]:[pathfrag]
```

Handlers can take an optional parameter list of data.  Data is specified as *key* = *value* pairs separated by commas (trailing commas are allowed).  Positional parameters are also allowed before any key/value pairs, and will be placed into a special data key named ```*args```.  Some handlers allow special '@'-key parameter values..

Here are some example paths, with and without arguments:
```
# simple path, no module, no arguments
/handler/path

# 'fetch' module, 'get' pathfrag, path defaults to '/'.
# 1 positional argument, 3 key-value arguments passed.  'x', and 'y' are normal, @method is a special parameter
/@fetch:get('/api/test', @method='post', x=10, y=20)

# no module, 2 positional parameters, one key-value parameter
/complex/run-handler(5, 10, userid=$.userid);

# can assign the return value from handlers to the data model
$.index = /api/get-index(partid=22)

# using the 'call' keyword, we can construct dynamic call paths
# arguments and return values work the same.
call "/test-handler" (x=10, @method='post');
call "/test" + $state.handlername, (x = $state.x, y = $state.y);
@rtnval = call "/test/handler";
```

Note that handlers are not *expressions*, so only simple assignment works, you can't write ```@foo = /test + 5```.

#### invalidate action

Invalidates (forces refresh) of &lt;h-data&gt; tags.  If no arguments are given, will invalidate
all &lt;h-data&gt; tags.  Otherwise takes a list of regular expressions to invalidate.

```
invalidate;
invalidate('query-1');
invalidate('^user-.*', '^customer-.*');
```

### fireevent action

Fire or bubble an event that triggers event handlers.  Parameters can be
name-value pairs which will create context (@vars) in the callee.  A single
positional parameter will set the 'value' key.

```
fire->click;
fire->custom1();
bubble->submit(itemid=22, price=100)
bubble->custom2;
bubble->custom3(88)
```

### throw action

Throws an error.  Takes one argument which will be evaluated as a string (the error message):
```
throw("Invalid User Id");
throw("No value is set");
```

### log action

Logs a sequence of positional arguments to the JavaScript console.  If @debug is true,
it will output extra debugging information and a Hibiki/JavaScript call stack.
If @alert is true, it will cause a browser alert.

```
log("the value is", $.value);
log("show stack trace", @debug=true);
log("Not Logged In!", @alert=true);
```

### if action

if/then/else implementation.  Parenthesis around the conditional expression,
Braces are required around the then/else statement blocks, else statement is optional.
If statements require a ```;``` to separate them from subsequent statements;

```
if ($.primary) { $.openpopup = true; };
if ($.color == 'blue') { $.blue = true } else { $.red = true };
```

### nop action

Does nothing.

```
nop;
nop();
```

#### expression action

Raw expressions can be statements.  This is only useful if your expression has
side effects.  Useful in conjunction with fn:js.  There is no "expr" JSON action,
instead you can use a "setdata" action with a null *setpath*.

```
expr fn:js('external_fn_name', 55);
expr fn:js('alert', 'hello');
```

