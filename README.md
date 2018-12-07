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

## Why

Many of us think that `globalThis` should not be added in the spec _as is_ in the current proposal. A lot of arguments has been made against it, so this is an attempt to fix it. 

_See https://github.com/tc39/proposal-global/issues/31 for the original issue raising the naming concerns._

## FAQ

##### Why a `GlobalReference` to expose the `globalThis`? Why we can't just expose it in e.g. `Object.global`, `Symbol.Global`, etc…?

_TL;DR;_
> Doing so would break compartment isolation and virtualization.

See https://github.com/tc39/proposal-global/issues/33 for more info on this.

##### How `GlobalReference` is different than `globalThis`?

As `global` breaks the web, many suggested to use e.g. `Global`; but the argument against it is (quoting https://github.com/tc39/proposal-global/issues/32):

> There was discussion of this, but it was thrown out because the pascal-cased names are supposed to be reserved for constructors and for namespaces. But when you think about the 99% use case for people, the "global reference" is always for namespacing reasons, and it is often referred to as the "global namespace". So to me it seems like an extremely relevant naming choice. This would preserve the greatest level of intent (since it's exactly the same word, just different capitalization) as the original name proposal. I think it may have been discarded too quickly.

Here, `GlobalReference` is actually a namespace.

##### Why the name `GlobalReference`?

Because _reference_ describes exactly what would be actually exposed, and it's a widely used term.

Also, the name is aligned with other proposals such as https://github.com/sebmarkbage/ecmascript-asset-references.
