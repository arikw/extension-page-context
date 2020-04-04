# Extension Page Context Runner

This utility was created to provide a utility for content scripts in web extensions (e.g.,  Chrome extensions) to run a function in the page context.
This is done by serializing the function to a string and injecting it to the web page - the returned value is then serialized using `JSON.stringify()` and transmitted back to the content script after unserializing it using `JSON.parse()`.

Usage examples -

```javascript

// Some code that exists only in the page context -
window.someProperty = 'property';
function someFunction(name = 'test') {
    return new Promise(res => setTimeout(()=>res('resolved ' + name), 1200));
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