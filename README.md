# But you promised 😢

[![CircleCI build status](https://img.shields.io/circleci/build/github/keirog/but-you-promised.svg)](https://circleci.com/gh/keirog/but-you-promised/tree/master) ![Node support](https://img.shields.io/node/v/but-you-promised.svg) [![MIT license](https://img.shields.io/badge/license-MIT-green.svg)](#license)

For when you don’t want your promises to give up on the first attempt (most commonly because of network failure).

Pass in a promise-returning function (X), get a wrapped function back that calls X until it fulfills/resolves, or until 5 attempts have been made. Exponential back-off by default, highly configurable, no dependencies.

## Contents

- [Syntax](#syntax)
- [Parameters](#parameters)
	- [Example custom back-off function](#example-custom-back-off-function)
- [Return value](#return-value)
- [Installation](#installation)
- [Usage](#usage)
	- [With promises](#with-promises)
	- [With async/await](#with-asyncawait)
- [Things to bear in mind](#things-to-bear-in-mind)
- [Migration guide](#migration-guide)
	- [Upgrading from v1.x.x to v2.x.x](#upgrading-from-v1xx-to-v2xx)
- [Licence](#licence)

## Syntax

`butYouPromised(yourFunction[, options])`

## Parameters

`yourFunction` _required function_

A function that returns [a promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). A common usecase would be a function that makes a network request when called.

`options` _optional object_

- `giveUpAfterAttempt` _optional integer, the default is 5_

	An integer that sets the maximum number of times `yourFunction` will be called before rejecting. The number set here will only ever be reached if your function’s promise consistently rejects.

- `createBackOffFunction` _optional function, the default creates an exponential delay function_

	A function used internally to create a back-off strategy between attempts, when first wrapping `yourFunction`. When called, `createBackOffFunction` should return a new function (let’s call it Y here for clarity). Y should return an integer and will be called after each failed attempt, in order to determine the minimum number of milliseconds to wait before another attempt (unless `giveUpAfterAttempt` has been reached). Y will be called internally with one parameter, which is a count of how many attempts have been made so far. This gives you flexibility to define how your subsequent attempts are made.

	### Example custom back-off function

	```js
	createBackOffFunction: ({ seedDelayInMs }) => {
		return attemptsSoFar => attemptsSoFar * seedDelayInMs;
	}
	```

	### If you don’t want a back-off

	```js
	createBackOffFunction: () => () => 0
	```

## Return value

Always returns a `function` that will return a promise when called.

## Installation

- via [npm](https://www.npmjs.com/get-npm): `npm install but-you-promised`
- via [yarn](): `yarn add but-you-promised`

## Usage

Wrap your promise-returning function like this:

```js
const { yourFunction } = require('./your-module');
const wrappedFunction = require('but-you-promised')(yourFunction);
```

### With promises

Before:

```js
yourFunction('yourParameter1', 'yourParameter2', 'yourParameter3')
	.then(result => console.log('Result:', result))
	.catch(() => {
		console.log('Failed after only 1 attempt');
	});
```

After:

```js
wrappedFunction('yourParameter1', 'yourParameter2', 'yourParameter3')
	.then(result => console.log('Result:', result))
	.catch(() => {
		console.log('Failed after a maximum of 5 attempts');
	});
```

### With async/await

Before:

```js
(async () => {
	try {
		const result = await yourFunction('yourParameter1', 'yourParameter2', 'yourParameter3');
		console.log('Result:', result);
	} catch (err) {
		console.log('Failed after only 1 attempt');
	}
}());
```

After:

```js
(async () => {
	try {
		const result = await wrappedFunction('yourParameter1', 'yourParameter2', 'yourParameter3');
		console.log('Result:', result);
	} catch (err) {
		console.log('Failed after a maximum of 5 attempts');
	}
}());
```

## Things to bear in mind

- It’s worth making sure that `yourFunction` doesn’t already make multiple attempts if a promise rejects (for example if you’re wrapping a third-party function), else you may make more network calls than you’re intending!
- If you’re using this software as part of an ongoing web request, consider using a custom back-off function (which delays exponentially by default), or reducing the default number of attempts (5), otherwise the original request may time out.

## Migration guide

### Upgrading from v1.x.x to v2.x.x

- Calling `butYouPromised` now returns a function wrapper for `yourFunction`
- No more passing your parameters in an awkward options.data object—use the returning function wrapper as you normally would use `yourFunction`
- optional overrides are passed in via an options object when wrapping `yourFunction`, where:
	-  `backoffStrategy` becomes `createBackOffFunction`
	- `triesRemaining` becomes `giveUpAfterAttempt`

```js
// v1 / Before
const yourParameterObj = { example: 123 };

butYouPromised(yourFunction, {
	backoffStrategy: ({ seedDelayInMs }) => {
		return attemptsSoFar => (attemptsSoFar * attemptsSoFar) * seedDelayInMs;
	},
	data: yourParameterObj,
	triesRemaining: 10
})
	.then(yourThenHandler)
	.catch(yourCatchHandler);
```
```js
// v2 / After
const yourParameterObj = { example: 123 };

const wrappedFunction = butYouPromised(yourFunction, {
	// If this is the back-off strategy you’re using, you can omit this now as it’s the default :-)
	createBackOffFunction: ({ seedDelayInMs }) => {
		return attemptsSoFar => (attemptsSoFar * attemptsSoFar) * seedDelayInMs;
	},
	giveUpAfterAttempt: 10
});

wrappedFunction(yourParameterObj)
	.then(yourThenHandler)
	.catch(yourCatchHandler);
```

## License

Published under the [MIT license](http://opensource.org/licenses/MIT).

