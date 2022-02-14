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

{{< toc >}}

## Specifying Data Paths

All data paths start with "$" or "@".  ```$``` is used for data roots
and ```@``` is for special contextual values.

* ```$``` - Accesses data from the data root (e.g. ```$.data.rows``` or ```$.state.open```).
* ```@``` - Binds to contextual data (e.g. ```@rownum``` or ```@value```).
* ```$c``` - Binds to component data (data local to a component)
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
* ```noattr``` - special null value that when passed as an attribute makes it appear that the attribute was not specified (see noattr/isnoattr below).
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
| **Comparison** | <=> | returns -1, 0, or 1, if first argument is less than, equal to, or greater than second element (respectively) |
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
| fn:floor | Math.floor |
| fn:ceil | Math.ceil |
| fn:slice | Like the JavaScript slice function.  First argument is an array, 2nd is start element (inclusive), 3rd element is the end element (exclusive). See [Array.slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice).<br>If you pass a named argument 'makerefs' it will return an array of *references* to the original array. |
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
| fn:uppercase | returns string value to uppercase |
| fn:lowercase | returns string value to lowercase |
| fn:match   | creates a regex with *arg2* and and *arg3* as options.  tries to match *arg1*.  <br>e.g. *arg1*.match(new Regexp(*arg2*, *arg3*)) |
| fn:uuid | creates a UUIDv4 |
| fn:json | converts its argument to JSON |
| fn:jsonparse | parses argument from JSON to an object |
| fn:bloblen | returns length of blob |
| fn:blobmimetype | returns blob mimetype |
| fn:blobastext | returns text version of blob (only if mimetype starts with "text/") |
| fn:blobasbase64 | returns base64 encoded version of blob |
| fn:blobname | returns the file name from the blob object |
| fn:deepequal | 2 arguments, return true/false if they are deeply equal |
| fn:deepcopy | creates a deepcopy of arguments (not usually necessary since Hibiki makes a copy automatically, except when dealing with references) |
| fn:compare | (see compare documentation below) |
| fn:sort | (see sort documentation below) |

### fn:compare()

Requires 2 arguments, the two values to be compared.  Returns -1, 0, or 1 if the first argument is less than, equal to, or greater than the 2nd argument (respectively).  The named parameters affect the comparison:

Named Parameters:

* **type** (string) - "numeric" or "string".  if null, the comparison will be based on the types of the arguments.  If both are numbers, then it will be a numeric comparison, otherwise string.  nulls will always sort before any value in either type.
* **locale** (string) - override the locale of the comparison (otherwise uses the browser's locale value).  e.g. for french should use "fr".
* **sensitivity** (string) - "base", "accent", "case", or "varient".  See [Intl Collator Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Collator/Collator).  By default it is set to "varient", but other sensitivity types allow different characters to be treated as equal.
* **nocase** - sets sensitivity to "accent" (sensitivity option takes precendence).  this makes the comparison case-insensitive.
* **field** - if both arguments are objects, compares the sub-field named *field* within them.  e.g. if you call fn:compare($.foo, $.bar, field='test'), it will compare $.foo.test with $.bar.test.  if argument is null, or does not contain the specified field, will compare against null.
* **index** - if both arguments are arrays, compares the sub-index *index* within them.  e.g. if you call fn:compare($.foo, $.bar, index=8), it will compare $.foo[8] with $.bar[8].  if argument is null, or does not contain the specified index, will compare against null.

```
fn:compare($.v1, $.v2, type="numeric")                # force numeric comparison
fn:compare($.v1, $.v2, type="string", nocase=true)    # force case-insensitive string comparison
fn:compare($.v1, $.v2, field="qty", type="numeric")   # numeric comparison between $.v1.qty and $.v2.qty

# compare $.v1.name with $.v2.name, using rules from french locale.
# comparison will be case-insensitive and will ignore accents
fn:compare($.v1, $.v2, field="name", locale="fr", sensitivity="base")
```

### fn:sort()

Requires 1 argument, the data to be sorted.  Returns a *new* array where the input data is sorted according to the provided arguments.

Named Parameters:

* **data** (array) - the data to be sorted.  can also be passed as the first positional argument.
* **sortexpr** (lambda) - a lambda expression that overrides the built in compare function.  Gets two context variables **@a** (first value), and **@b** (second value).  Should return -1, 0, or 1.
* **desc** (boolean) - if true, will reverse the sort order
* **makerefs** (boolean) - if true, will create a sorted array of references that point back to the original array elements.
* **slice** ([number, number]) - a one or two element array to slice the final array.  same arguments as fn:slice().
* *(any parameter that can be passed to fn:compare)* - these parameters are passed to fn:compare() when comparing values.  not used if *sortexpr* is provided.

```
fn:sort($.data)                               // sorts data naturally using fn:compare() function
fn:sort($.data, field="qty", type="numeric")  // sort objects using field "qty" numerically
fn:sort($.data, makerefs=true)                // returns array of references to original array
fn:sort($.data, field="qty", desc=true)       // sort objects using field "qty", reverse order
fn:sort($.data, slice=[10,20])                // sort data naturally, return elements 10-19
```

## Special Expressions

| Expression | Notes |
|------------|-------|
| ref(*path-expr*) | creates a reference to the given path. can only accept global or component paths ($) or ($c). |
| isref(*expr*)    | returns true/false if the expression is a reference |
| refinfo(*expr*)  | if expr is a reference returns a string showing what it references, otherwise null |
| raw(*expr*) | normally references are automatically de-referenced (so they are transparent).  using the expression raw() will keep them from being evaluated.  useful for passing references to a sub-component. |
| noattr | the special constant meaning "no attribute".  generally in most user expressions it is equivalent to null.  however for some components passing an argument equal to "null" or not passing any argument is different.  The "noattr" expression allows you to propagate the "no attribute" to a sub-component. |
| isnoattr(*expr*) | returns true/false if the expr is equal to "noattr".  you cannot distinguish "noattr" with "==" because noattr == null (and vice versa). |
| lambda(*expr*)   | creates a special lambda value of the given expression.  lambda values are not evaluated until they are invoked.  They will only have access to the context data that is passed in during invoke.  Any other data references (e.g. to global data, component data, or current context data) will be undefined. |
| invoke(*lambda*, *params*) | invokes a lambda.  params are context variables to be passed into the expression |

## Special Data Roots

| Data Root | Notes |
|-----------|-------|
| $ | Alias for $data (global data root) |
| @ | Alias for $context |
| $c | Component root data (data local to the current component) - alias for $component |
| $data | Returns root of frontend data model (global root) |
| $component | Component root data (data local to the current component)
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

