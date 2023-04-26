# jest-webgl-canvas-mock

> Mock `canvas` and `WebGL` when running unit test cases with jest.

[![Build Status](https://travis-ci.org/adamfsk/jest-webgl-canvas-mock.svg?branch=master)](https://travis-ci.org/adamfsk/jest-webgl-canvas-mock)
[![Coverage Status](https://coveralls.io/repos/github/adamfsk/jest-webgl-canvas-mock/badge.svg?branch=master)](https://coveralls.io/github/adamfsk/jest-webgl-canvas-mock)
[![npm](https://img.shields.io/npm/v/jest-webgl-canvas-mock.svg)](https://www.npmjs.com/package/jest-webgl-canvas-mock)
[![npm](https://img.shields.io/npm/dm/jest-webgl-canvas-mock.svg)](https://www.npmjs.com/package/jest-webgl-canvas-mock)

## Disclaimer

This project is a simple merge of [jest-canvas-mock](https://github.com/hustcc/jest-canvas-mock) with [webgl-mock](https://github.com/kbirk/webgl-mock) so that both 2d and webgl contexts can be tested in jest. As such, the only tests provided are those from the original projects. 

The current goal of this project is simply to make any tests using `pixi.js` work in jest.

Please feel free to contribute and add any additional functionality required.

## Install

This should only be installed as a development dependency (`devDependencies`) as it is only designed for testing.

```bash
npm i --save-dev jest-webgl-canvas-mock
```


## Setup

In your `package.json` under the `jest`, create a `setupFiles` array and add `jest-webgl-canvas-mock` to the array.

```json
{
  "jest": {
    "setupFiles": ["jest-webgl-canvas-mock"]
  }
}
```

If you already have a `setupFiles` attribute you can also append `jest-webgl-canvas-mock` to the array.

```json
{
  "jest": {
    "setupFiles": ["./__setups__/other.js", "jest-webgl-canvas-mock"]
  }
}
```

More about in [configuration section](https://facebook.github.io/jest/docs/en/configuration.html#content).


## Setup file

Alternatively you can create a new setup file which then requires this module or
add the `require` statement to an existing setup file.

`__setups__/canvas.js`

```js
import 'jest-webgl-canvas-mock';
// or
require('jest-webgl-canvas-mock');
```

Add that file to your `setupFiles` array:

```json
"jest": {
  "setupFiles": [
    "./__setups__/canvas.js"
  ]
}
```

## Mock Strategy

This mock strategy implements all the canvas functions and actually verifies the parameters. If a
known condition would cause the browser to throw a `TypeError` or a `DOMException`, it emulates the
error. For instance, the `CanvasRenderingContext2D#arc` function will throw a `TypeError` if the
radius is negative, or if it was not provided with enough parameters.

```ts
// arc throws a TypeError when the argument length is less than 5
expect(() => ctx.arc(1, 2, 3, 4)).toThrow(TypeError);

// when radius is negative, arc throws a dom exception when all parameters are finite
expect(() => ctx.arc(0, 0, -10, 0, Math.PI * 2)).toThrow(DOMException);
```

The function will do `Number` type coercion and verify the inputs exactly like the browser does. So
this is valid input.

```ts
expect(() => ctx.arc("10", "10", "20", "0", "6.14")).not.toThrow();
```

Another part of the strategy is to validate input types. When using the
`CanvasRenderingContext2D#fill` function, if you pass it an invalid `fillRule` it will throw a
`TypeError` just like the browser does.

```ts
expect(() => ctx.fill("invalid!")).toThrow(TypeError);
expect(() => ctx.fill(new Path2D(), "invalid!")).toThrow(TypeError);
```

We try to follow the ECMAScript specification as closely as possible.

# Snapshots

There are multiple ways to validate canvas state. There are currently three `static` methods attached
to the `CanvasRenderingContext2D` class. The first way to use this feature is by using the `__getEvents`
method.

```ts
/**
 * In order to see which functions and properties were used for the test, you can use `__getEvents`
 * to gather this information.
 */
const events = ctx.__getEvents();

expect(events).toMatchSnapshot(); // jest will assert the events match the snapshot
```

The second way is to inspect the current path associated with the context.

```ts
ctx.beginPath();
ctx.arc(1, 2, 3, 4, 5);
ctx.moveTo(6, 7);
ctx.rect(6, 7, 8, 9);
ctx.closePath();

/**
 * Any method that modifies the current path (and subpath) will be pushed to an event array. When
 * using the `__getPath` method, that array will sliced and usable for snapshots.
 */
const path = ctx.__getPath();
expect(path).toMatchSnapshot();
```

The third way is to inspect all of the success draw calls submitted to the context.

```ts
ctx.drawImage(img, 0, 0);

/**
 * Every drawImage, fill, stroke, fillText, or strokeText function call will be logged in an event
 * array. This method will return those events here for inspection.
 */
const calls = ctx.__getDrawCalls();
expect(calls).toMatchSnapshot();
```

## Override default mock return value

You can override the default mock return value in your test to suit your need. For example, to override return value of `toDataURL`:

```ts
canvas.toDataURL.mockReturnValueOnce(
  'data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=='
);
```

## License

MIT
