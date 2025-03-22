# JSON Style Guide

This style guide documents guidelines and recommendations for building JSON APIs for WSO. In general, JSON APIs should follow the spec found at JSON.org. This style guide simply clarifies and standardizes specific cases so that all JSON APIs have a standard look and feel.

## Property Name Format
Choose meaningful property names.

Property names must conform to the following guidelines:

* Property names should be meaningful names with defined semantics.
* Property names must be _camelCase_, ascii strings.
* The first character must be a letter, an underscore (\_) or a dollar sign ($).
* Subsequent characters can be a letter, a digit, an underscore, or a dollar sign.
* Reserved JavaScript keywords should be avoided.

```json
{
  "thisPropertyIsAnIdentifier": "identifier value"
}
```

### camelCase Initialisms

Words in names that are initialisms or acronyms (e.g. "URL" or "NATO") have a consistent case. For example, "URL" should appear as "URL" or "url" (as in "urlPony", or "URLPony"), never as "Url". As an example: ServeHTTP not ServeHttp. For identifiers with multiple initialized "words", use for example "xmlHTTPRequest" or "XMLHTTPRequest".

This rule also applies to "ID" when it is short for "identifier" (which is pretty much all cases when it's not the "id" as in "ego", "superego"), so write "appID" instead of "appId".

## API Response Guidelines

An API response will have three properties:

* `status` - an integer describing the HTTP status the server returned
* `data` - the data payload sent from the server
* `error` - an API error object

### API Error

The API Error object has two properties:
* `errorCode` - an integer describing the error. If the value is `< 1000`, the error code is an HTTP error code, otherwise it is an API-specific error code.
* `message` - the error message as a string
