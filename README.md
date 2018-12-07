# proposal-global-reference

An alternative to https://github.com/tc39/proposal-global

You can copy and run this code in your browser console or Node.js.

```js
// a naive **internal** implementation example
// i.e. this would be actually implemented by JS engines
class Global {
  static [Symbol.hasInstance](instance) {
    return false;
  }
  constructor() {
    // this is not possible in userland due to CSP policy e.g. chrome apps
    // see https://developer.chrome.com/apps/contentSecurityPolicy
    this.globalThis = Function('return this')();
    // whatever that would need to be further exposed? e.g.
    this.hostEnvironment = 'browser';
    // …
  }
}

// exposed intrinsic object
GlobalReference = Object.setPrototypeOf(new Global(), null);

// tests
console.assert(GlobalReference != window);
console.assert(window.environment == null);
console.assert(GlobalReference.hostEnvironment);
console.assert(GlobalReference.globalThis === window);
console.assert(GlobalReference instanceof Function === false);
console.assert(GlobalReference instanceof Global === false);
console.assert(GlobalReference instanceof Object === false);
console.assert(GlobalReference.prototype === undefined);
console.assert(GlobalReference.__proto__ === undefined);

// userland
const userland = {
  script() {
    console.log('local `this`', this);
    console.log('global `this`', GlobalReference.globalThis);
    return `That's it.`;
  }
};

userland.script();
```
