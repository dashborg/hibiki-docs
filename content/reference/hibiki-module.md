---
title: Hibiki Module
weight: 10
---

{{< toc >}}

The **hibiki** module is used for misc functionality that accesses some built in browser functionality.
To call a hibiki module function use `//@hibiki/[fn-name](args)`.

| Handler | Example | Description |
|---------|---------|-------------|
| /sleep  | `//@hibiki/sleep(100)` | sleeps for given number of ms |
| /alert  | `//@hibiki/alert("hello")` | calls the browser alert function with the given argument |
| /confirm | `@ok = //@hibiki/confirm("Are you sure?")` | calls the browser confirm function with the given argument.  returns true/false |
| /set-session-storage | `//@hibiki/get-session-storage(key='test')` | retrieves the given session storage key, value will be parsed as JSON and returned as an object.  if storage is not valid JSON will return null. |
| /get-session-storage | `//@hibiki/set-session-storage(key='test', value='hello') `| sets the given session storage key to the given value.  value will be converted to JSON (error if it cannot be converted). if value is 'null' the key will be removed. |
| /set-title | `//@hibiki/set-title('New Title')` | Sets the page title in browser (equivalent to calling document.title = value) |
| /update-url | `//@hibiki/update-url(@path='/test', @title="New Title", @replace=false, arg1=10)` | Updates the browser URL to @path (using HTML5 history API, does not navigate the browser).  calls pushState by default, or replaceState if @replace=true.  @title is optional, and will set document.title if given. Non @-args will be set as parameters on the given @path. |
| /navigate | `//@hibiki/navigate(@path='/test', arg1=10, y=20)` | will navigate the browser to the given @path (by setting window.location).  the non-@-args will be set as query parameters on the URL. |
| /set-timeout | `//@hibiki/set-timeout("tevent", 1000, arg=5, y=10)` | Calls the given event after the given number of milliseconds.  The named arguments are passed as event arguments |
| /set-interval | `@id = //@hibiki/set-interval("ievent", 1000, arg=5, y=10)` | Calls the given event every N milliseconds.  The named arguments are passed as event arguments.  Will call forevery until cleared using clear-interval with the returned id |
| /clear-interval | `//@hibiki/clear-interval(@id)` | Stops a set-interval from firing (also works with unfired set-timeouts).  Must pass the id that is returned from set-interval or set-timeout |
