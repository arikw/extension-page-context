# Extension Page Context Runner

## The "problem"

When building a web extension (e.g.,  Chrome extension) you'd sometimes want to interact with scripts that run inside a webpage.

As stated [on MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_scripts#Content_script_environment):

> ...content scripts get a "clean" view of the DOM. This means:
>
> - Content scripts cannot see JavaScript variables defined by page scripts.
> - If a page script redefines a built-in DOM property, the content script sees the original version of the property, not the redefined version

**Meaning content scripts run in a different context than the context of page scripts**

So if, for example, you'd want to overwrite the `onClick` property of a DOM element in a content script, you will end up with two event handlers - one is the original handler and the second is the handler defined in the content script.

To solve this issue, your script must run as a "page script" by injecting code into a `<script>` element.

## The "Extension Page Context"

This utility facilitates the following -
- Running code directly in the page's context
- Getting the return value back to the content script

### Usage

Include `page-context.js` file in your project and call `runInPageContext()`, passing it a "script function" or a config object like so:

```javascript
runInPageContext(scriptFn /* script function to inject */);
// or
runInPageContext({
    func: scriptFn,  // script to inject
    args: [],        // arguments for the script function
    doc: document,   // script injection destination
    timeout: 10000   // timeout
});
```

Finally `runInPageContext` returns a `Promise`.

The `scriptFn` will be injected into the page and get executed as a page script.  
The resolved value of `scriptFn` will be returned back to the content script as follows:
- If `scriptFn` returns a `Promise`, the resolved value (or rejection) will be used to resolve the `Promise` returned by `runInPageContext`
- Otherwise, the returned value will be used to resolve the `Promise` returned by `runInPageContext`.

*Notice that the resolved value will be serialized (`JSON.stringify()`) before being passed from the page context to the content script context and unserialized (`JSON.parse()`) afterwards.*

## Usage Examples

```javascript

// Some code that exists only in the page context -
window.someProperty = 'property';
function someFunction(name = 'test') {
    return new Promise(
        res => setTimeout( () => res('resolved ' + name), 1200 )
    );
}
/////////////////

// Content script examples -

await runInPageContext(() => someProperty); // returns 'property'

await runInPageContext(() => someFunction()); // returns 'resolved test'

await runInPageContext(async (name) => someFunction(name), 'with name' ); // 'resolved with name'

await runInPageContext(async (...args) => someFunction(...args), 'with spread operator and rest parameters' ); // returns 'resolved with spread operator and rest parameters'

await runInPageContext({
    func: (name) => someFunction(name),
    args: ['with params object'],
    doc: document,
    timeout: 10000
} ); // returns 'resolved with params object'


```

## More information

The passed function is serialized to a string and injected into a temporary `<script>` element added to the web page.  
The resolved value of the function is serialized and transmitted back to the content script using the messsaging mechanism.
