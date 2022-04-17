---
title: Http Module
---

{{< toc >}}

The **http** module is used for all basic AJAX calls in Hibiki HTML.  It allows you
to call AJAX handlers and returns their results.

{{<hint info>}}
The module is named **http**, but it handles both **http** and **https** requests!
{{</hint>}}

Here are some sample http module calls:

```
  # simple GET
  GET /api/test-1
  
  # a relative URL
  GET relative/url.html?test=1
  
  # POST to a remote resource
  POST https://remoteserver.any.com/api/test-post
  
  # 'GET' can be omitted for URLs that start with http, https, or //
  https://testapi.hibikihtml.com/api/random-number
  
  # adding a parameter x=55, will be added to URL
  # e.g. equivalent to /api/test-1?y=22&x=55
  GET /api/test-1?y=22(x=55)
  
  # POST, with url encoded parameters
  POST /api/test-post(user={id: 55, name: 'mike'}, access_code='A55', @enc='url')
```

Each call can be configured by passing special ```@``` parameters.  Global configuration
is done through the ```httpConfig``` key of the global &lt;hibiki-config&gt;.

Sometimes you need special http options for a particular host (e.g. a REST endpoint that
requires special headers or CSRF handling).  You can set that up easily by adding an
addition HTTP module (see Additional HTTP Modules below).

## Parameters to Handler Calls

Any non ```@``` parameter is considered data and will be passed to the AJAX handler.  Here are
the special parameters that all http calls accept.  See the [fetch() API documentation](https://developer.mozilla.org/en-US/docs/Web/API/fetch).

| Parameter | Type | Description |
|-----------|------|-------------|
| @method   | string | Overrides the http method for this request. e.g. @method='POST'|
| @encoding | string | Sets the encoding for this request.  Defaults to 'json'.  Other valid values are 'url' for application/x-www-form-urlencoded, or 'multipart' for multipart/form-data. |
| @data     | object | To pass data an an object. e.g. @data={x: 55, color: "blue"}.  Individual params set will override what is specified in data.  e.g. (@data={x: 55, y:5}, x=22) will set x to 22 and y to 5. |
| @csrf     | string | Set the CSRF token for this request.  Overrides the value that would have been produced by the default config (csrfToken). Note that the rest of the CSRF checks will still apply (csrfAllowedOrigins and csrfMethods) so specifiying @csrf does not guarantee a CSRF token will be sent. |
| @headers  | object | An object with header names as keys, and values as header values.  Used to set additional headers on the request.  e.g. @headers={"X-User-Id": $.userid}. |
| @mode | string | Overrides the fetch 'mode'.  'cors', 'no-cors', or 'same-origin'|
| @credentials | string | Overrides the fetch 'credentials'.  'omit', 'same-origin', or 'include'. |
| @init | string | Sets the default fetchInit object.  This is the base object that will be used, and then additional '@' params and configuration defaults will be applied.  This can be used to set uncommon values that are not covered by the other '@' params, e.g. 'cache', 'redirect', 'referrer', 'referrerPolicy', 'integrity', 'keepalive', etc. |

These '@' params are used for one-off overrides of individual requests.  If you always want to send a particular
header or set a special init setting, it can often be easier to specify the value in the HttpConfig object (see below).

## Dynamic URLs

To make an AJAX handler call with a dynamic URL, you need to use the ```callhandler``` statement.  It
takes the same parameters (above), but additionally it takes two more ```@url``` and ```@module```:

| Paramter | Type   | Description |
|----------|--------|-------------|
| @url     | string | the URL to fetch (passed to the module), required |
| @module  | string | the module to use, defaults to the 'http' module (which also handles 'https' calls) |

Here's some examples:

```
callhandler(@url="https://testapi.hibikihtml.com/api/get-color-1");

@handler_name = 'get-color-' + $.colornum;
callhandler(@url="https://testapi.hibikihtml.com/api/" + @handler_name)

# hibiki module call, equivalent to //@hibiki/sleep(ms=1000);
callhandler(@module="hibiki", @url="/sleep", ms=1000);
```


## Http Config

Http Config is set as the ```httpConfig``` key of the global &lt;hibiki-config&gt;:

```
<template hibiki>
  <hibiki-config>
  {
     ... other config options ...
     "httpConfig": { ... http options ... }
  }
  </hibiki-config>
</template>
```

| Key | Type | Description |
|-----|------|-------------|
| baseUrl | string | Relative URLs will be made relative against the specified URL.  If not specified, relative URLs will be built against window.location.href. |
| lockToBaseOrigin | boolean | If set to ```true```, will throw an error if trying to fetch an URL that has a different origin from the specified baseUrl (or window.location.href if baseUrl is not set). |
| forceRelativeUrls | boolean | If set to ```true```, will throw an error if trying to fetch an URL which is not relative to the baseUrl.  e.g. if baseUrl is 'https://hibikihtml.com/test/', an url with a different host name, protocol, port, or that does not begin with '/test/' would be rejected. |
| defaultHeaders | headerName -> actionStr | Custom headers to be added to every request. See below for more details. |
| defaultInit | object | Default init object to be passed to fetch.  see [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/fetch) |
| csrfToken | string or actionString | CSRF token for requests.  Can be specified as a literal string, or as a hibiki action string (to dynamically pull it from the data model) |
| csrfMethods | string[] | an array of HTTP methods to pass the CSRF token to.<br>defaults to ["POST", "PUT", "PATCH"] |
| csrfParams | string[] | if a CSRF token is found, an array of parameter names to set with the token's value.  Defaults to [].  e.g. can set to ["_token"] for a framework like Laravel. |
| csrfHeaders | string[] | if a CSRF token is found, an array of header names to set with the token's value.  Defaults to ["X-CSRF-Token", "X-CSRFToken"].  This is the default for many backend libraries. |
| csrfAllowedOrigins | string[] | Restricts the CSRF token from being sent to only the listed origins.  Defaults to the baseUrl's origin.  Can set to ["*"] to send to all origins. |
| fetchHookFn | hookFn | A hook function that runs right before fetch() is called.  ((url : URL, fetchInit : any) => void).  Can modify url or fetchInit.  To cancel the request it can throw an exception. |

## Additional HTTP Modules

If you are making requests to multiple endpoints which require different headers, CSRF settings, or fetchInit parameters, you can configure additional HTTP modules.

Here's an example config that sets up a module named 'app' that always includes an 'X-UserId' header set to a hibikiexpr of '$.userid', a static header (X-Domain-Id), and a special CSRF configuration.  Notes:

* The name of the module is the key under "modules", in this case "app".
* The "type" key is required.  For HTTP modules it must be set to "http".
* The rest of the configuration matches the HttpConfig section above.
* For special modules that are for specific API endpoints, you should set "forceRelativeUrls" to only allow allow this configuration for requests to the specific origin / subpath.

```
<hibiki-config>
{
  "modules": {
      "app": {"type": "http",
              "baseUrl": "https://my.api.com/api/v2/",
              "forceRelativeUrls": true,
              "defaultHeaders": {
                  "X-UserId": {"hibikiexpr": "$.userid"},
                  "X-Domain-Id": "81288642"
              },
              "csrfToken": {"hibikiexpr": "$.apitoken"},
              "csrfParams": ["_token"]
      }
  }
}
</hibiki-config>
```

You can now use the 'app' module to make requests to "my.api.com" with the given HTTP settings:

```
# makes a GET request to https://my.api.com/api/v2/get-data
GET //@app/get-data(id=22);

# makes a POST request to https://my.api.com/api/v2/primary-name/set-name
POST //@app/primary-name/set-name(name='mike');
```
