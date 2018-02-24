---
layout: post
title: Implement Promise In ES6
key:   20170915
tags: Promise ES6 Journal
---

We had been using async / await for our project for a while, our new staff seemed to have not trouble with implementation of async programming as I did. But still, I believe Promise is essential for js programming, not only async function return Promise, but also cutting edge technique such WebAssembly highly replied on it.

This is as simple journal for implementation Promise in terms of Promise/A++ standard[^1] which is the same standard of ES6 Promise.

### Analyse

1. Promise have three states: pending, fulfilled, rejected
2. Promise have a then function 
3. Then function return a promise
4. Different Promise implementations are interoperable

### Promise Status

> 2.1 A promise must be in one of three states: pending, fulfilled, or rejected.[^1]
> * When pending, a promise:
> 	* may transition to either the fulfilled or rejected state.


```js

const STATUS = {
  PENDING: 0,
  FULFILLED: 1,
  REJECTED: 2
}

class Promise {
  constructor (exec) {
    let status = STATUS.PENDING
    let value = undefined
  }
}

```

Now, we have a simple promise 

```js
let somePromise = new Promise(function(resolve, reject) {

/*
* When fulfilled call resolve, a promise:
  * must not transition to any other state.
  * must have a value, which must not change.

* When rejected call reject, a promise:
  * must not transition to any other state.
  * must have a reason, which must not change.
*/

})
```

Then we need add two function parameter to exec function: resole and reject


```js
class Promise {
  constructor (exec) {
    this.status = STATUS.pending
	this.value = undefined
	function resolve(value) {
		if (this.status === STATUS.PENDING) {
			this.status = STATUS.FULFILLED;
			this.value = value;
		}
	}
	function reject(error) {
		if (this.status === STATUS.PENDING) {
			this.status = STATUS.REJECTED;
			this.value = error;
		}		
	}
	exec(resolve, reject)
  }
}
```

The code above simple implement Promise / A++ 2.1[^1], however there are three issues here:


* status and value is not private and can be changed outside
* this issue which vary in contexts
* exec could failed eg ``` new Promise((resolve, reject) => { throw 1}) ```

```js
// use Symbol to encapsulate private members

const _status = Symbol('status')
const _value = Symbol('value)

class Promise {
  constructor (exec) {
	// bind self to this
	let self = this
    self[_status] = STATUS.pending
	self[_value] = undefined
	function resolve(value) {
		if (self[_status] === STATUS.PENDING) {
			self[_status] = STATUS.FULFILLED;
			self[_value] = value;
		}
	}
	function reject(error) {
		if (self[_status] === STATUS.PENDING) {
			self[_status] = STATUS.REJECTED;
			self[_status] = error;
		}		
	}
	try {
		exec(resolve, reject)
	} catch (e) {
		reject(e)
	}
	
  }
}
```
### Then method

> A promise must provide a then method to access its current or eventual value or reason.
> A promise’s then method accepts two arguments:
> 	```promise.then(onFulfilled, onRejected)```
> Both onFulfilled and onRejected are optional arguments:
> * If onFulfilled is not a function, it must be ignored.
> * If onRejected is not a function, it must be ignored.

```js

class Promise {
	constructor (exec) {
		// ...
	}
	then(onFulfilled, onRejected) {
		let self = this
		// make sure onFulfilled and onRejected be functions
		onFulfilled = typeof (onFulfilled) === 'function' ? onFulfilled : (value) => {}
        onRejected = typeof (onRejected) === 'function' ? onRejected : (reason) => {}
	}
}
```

> 2.2.1 If onFulfilled is a function:
> * it must be called after promise is fulfilled, with promise’s value as its first argument.
> * it must not be called before promise is fulfilled.
> * it must not be called more than once.

> 2.2.2 If onRejected is a function,
> * it must be called after promise is rejected, with promise’s reason as its first argument.
> * it must not be called before promise is rejected.
> * it must not be called more than once.


```js

if (self[_status] === STATUS.FULFILLED) {

}

if (self[_status] === STATUS.REJECTED) {

}

if (self[_status] === STATUS.PENDING) {

}
```

> 2.2.4 onFulfilled or onRejected must not be called until the **execution context stack** contains only platform code. [3.1].
> 2.2.5 onFulfilled and onRejected must be called as functions (i.e. with no this value).
> 3.1 Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that onFulfilled and onRejected execute asynchronously, after the event loop turn in which then is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as setTimeout or setImmediate, or with a “micro-task” mechanism such as MutationObserver or process.nextTick. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.
> 3.2 That is, in strict mode this will be undefined inside of them; in sloppy mode, it will be the global object

```js

const nextTick = process && process.nextTick : process.nextTick ? setTimeout

	function resolve(value) {
		nextTick (() => {
			if (self[_status] === STATUS.PENDING) {
				self[_status] = STATUS.FULFILLED;
				self[_value] = value;
			}
		})
	}
	function reject(error) {
		nextTick (() => {
			if (self[_status] === STATUS.PENDING) {
				self[_status] = STATUS.REJECTED;
				self[_value] = error;
			}		
		})
	}
```


> 2.2.6 then may be called multiple times on the same promise.
> *	If/when promise is fulfilled, all respective onFulfilled callbacks must execute in the order of their originating calls to then.
> * If/when promise is rejected, all respective onRejected callbacks must execute in the order of their originating calls to then.

OK, we have two new concepts here: onFulfiled callbacks and onRejects callbacks. 

```js
const _onFulfilledCallbacks = Symbol('onFulfilledCallbacks')
const _onRejectedCallbacks= Symbol('onRejectedCallbacks)
class Promise {
  constructor (exec) {
	// bind self to this
	let self = this
    self[_status] = STATUS.pending
	self[_value] = undefined
	self[_onFulfilledCallbacks] = []
	self[_onRejectedCallbacks] = []
	function resolve(value) {
		nextTick (() => {
			if (self[_status] === STATUS.PENDING) {
				self[_status] = STATUS.FULFILLED;
				self[_value] = value;
				self[_onFulfilledCallbacks].map(cb => cb(value))
			}
		})
	}
	function reject(error) {
		nextTick (() => {
			if (self[_status] === STATUS.PENDING) {
				self[_status] = STATUS.REJECTED;
				self[_value] = error;
				self[_onRejectedCallbacks].map(cb => cb(error))
			}	
		})	
	}
	try {
		exec(resolve, reject)
	} catch (e) {
		reject(e)
	}
	
  }
}
```

> 2.2.7 then must return a promise
>	```promise2 = promise1.then(onFulfilled, onRejected)```
> * 2.2.7.1 If either onFulfilled or onRejected returns a value x, run the Promise Resolution Procedure [[Resolve]](promise2, x).
> * 2.2.7.2 If either onFulfilled or onRejected throws an exception e, promise2 must be rejected with e as the reason.
> * 2.2.7.3 If onFulfilled is not a function and promise1 is fulfilled, promise2 must be fulfilled with the same value as promise1.
> * 2.2.7.4 If onRejected is not a function and promise1 is rejected, promise2 must be rejected with the same reason as promise1.

This is the most tricky part in promise implementation, let us define a resolver for [[Resolve]](promise2, x
)

let us code it step by step

1. then must return a promise


```js

then(onFulfilled, onRejected) {
	let self = this
	let promise2
	// make sure onFulfilled and onRejected be functions
	onFulfilled = typeof (onFulfilled) === 'function' ? onFulfilled : (value) => {}
	onRejected = typeof (onRejected) === 'function' ? onRejected : (reason) => {}

	if (self[_status] === STATUS.FULFILLED) {
		return promise2 = new Promise((resolve, reject) => {

		})
	}

	if (self[_status] === STATUS.REJECTED) {
		return promise2 = new Promise((resolve, reject) => {
			
		})
	}

	if (self[_status] === STATUS.PENDING) {
		return promise2 = new Promise((resolve, reject) => {
			
		})
	}
}
```

2. either onFulfilled or onRejected returns a value x, run the Promise Resolution Procedure [[Resolve]](promise2, x).

Add resolve constrain for [[Resolve]](promise2, x) 

```js
resolver (promise2, x) {

}

then(onFulfilled, onRejected) {
// ...

	if (self[_status] === STATUS.FULFILLED) {
		return promise2 = new Promise((resolve, reject) => {
			nextTick(() => {
				let x = onFulfilled(self[_value])
				resolver(promise2, x)
			})
		})
	}

	if (self[_status] === STATUS.REJECTED) {
		return promise2 = new Promise((resolve, reject) => {
			nextTick(() => {
				let x = onRejected(self[_value])
				resolver(promise2, x)
			})			
		})
	}
}

```

3. If either onFulfilled or onRejected throws an exception e, promise2 must be rejected with e as the reason.

```js
// ...
	if (self[_status] === STATUS.FULFILLED) {
		return promise2 = new Promise((resolve, reject) => {
			nextTick(() => {
				try {
					let x = onFulfilled(self[_value])
					resolver(promise2, x)
				} catch (e) {
					reject(e)
				}
			})
		})
	}

	if (self[_status] === STATUS.REJECTED) {
		return promise2 = new Promise((resolve, reject) => {
			nextTick(() => {
				try {
					let x = onRejected(self[_value])
					resolver(promise2, x)
				} catch (e) {
					reject(e)
				}

			})			
		})
	}	
```

4. If onFulfilled is not a function and promise1 is fulfilled, promise2 must be fulfilled with the same value as promise1. Or if onRejected is not a function and promise1 is rejected, promise2 must be rejected with the same reason as promise1

In other words, eg

```js
// should console 'aa'
promise1
.then(resolve('aa))
.then()
.then()
.then((v) => console.log(v) )
```

Which means default onFulfilled or onReject should pass the value along the chain

```js
onFulfilled = typeof (onFulfilled) === 'function' ? onFulfilled : (value) => value
onRejected = typeof (onRejected) === 'function' ? onRejected : (reason) => reason
```


For now we have not implement pending status, because there is no way to determine the final status to be fulfilled or rejected. We need to cache the callbacks to previous promise callbacks array

```js
// ...
	if (self[_status] === STATUS.PENDING) {
		return promise2 = new Promise((resolve, reject) => {
			self[_onFulfilledCallbacks].push(() => {
				try {
					let x = onFulfilled(self[_value])
					resolver(promise2, x)
				} catch (e) {
					reject(e)
				}
			})

			self[_onRejectedCallbacks].push(() => {
				try {
					let x = onRejected(self[_value])
					resolver(promise2, x)
				} catch (e) {
					reject(e)
				}
			})			
		})
	}
```

### The Promise Resolution Procedure

> 2.3.1 If promise and x refer to the same object, reject promise with a TypeError as the reason.

We need also pass resolve and reject methods of promise2 as parameters

```js

resolver (promise2, x, resolve, reject) {
	if (promise2 === x) {
		return reject(new TypeError('Same promise objects'))
	}
}
```

> 2.3.2 If x is a promise, adopt its state [3.4]:
> * If x is pending, promise must remain pending until x is fulfilled or rejected.
> * If/when x is fulfilled, fulfill promise with the same value.
> * If/when x is rejected, reject promise with the same reason.

```js

resolver (promise2, x, resolve, reject) {
	if (promise2 === x) {
		return reject(new TypeError('Same promise objects'))
	}

	if (result instanceof Promise) {
		if (result[_status] === STATUS.PENDING) {
			result.then(v => resolver(promise, v, resolve, reject), reject)
		} else {
			result.then(resolve, reject)
		}
	}
}
```

> 2.3.3 Otherwise, if x is an object or function
> * 2.3.3.1 Let then be x.then. [3.5]
> 3.5 This procedure of first storing a reference to x.then, then testing that reference, and then calling that reference, avoids multiple accesses to the x.then property. Such precautions are important for ensuring consistency in the face of an accessor property, whose value could change between retrievals.
> * 2.3.3.2 If retrieving the property x.then results in a thrown exception e, reject promise with e as the reason.
> * 2.3.3.3 If then is a function, call it with x as this, first argument resolvePromise, and second argument rejectPromise, where:
> 	*	2.3.3.3.1 If/when resolvePromise is called with a value y, run [[Resolve]](promise, y).
> 	*	2.3.3.3.2 If/when rejectPromise is called with a reason r, reject promise with r.
>	* 	2.3.3.3.3 If both resolvePromise and rejectPromise are called, or multiple calls to the same argument are made, the first call takes precedence, and any further calls are ignored.
>	*	2.3.3.3.4 If calling then throws an exception e,
>		* 	2.3.3.3.4.1 If resolvePromise or rejectPromise have been called, ignore it.
>		* 	2.3.3.3.4.2 Otherwise, reject promise with e as the reason.
> * 2.3.3.4 If then is not a function, fulfill promise with x.
> 2.3.4 If x is not an object or function, fulfill promise with x.

```js

resolver (promise2, x, resolve, reject) {
	let then, thenCalledOrThrow = false 
	if (promise2 === x) {
		return reject(new TypeError('Same promise objects'))
	}

	if (x instanceof Promise) {
		if (x[_status] === STATUS.PENDING) {
			x.then(v => resolver(promise, v, resolve, reject), reject)
		} else {
			x.then(resolve, reject)
		}
	}

	if ((x !== null) && (typeof (x) === 'object' || typeof (x) === 'function')) {
		try {
			then = x.then // 2.3.3.1
			if (typeof (then) === 'function') { // 2.3.3.3
				then.call(x, s => { // 2.3.3.3.1
					if (thenCalledOrThrow) return;
					thenCalledOrThrow = true;
					return resolver(promise, s, resolve, reject);
				}, r => { // 2.3.3.3.2
					if (thenCalledOrThrow) return;
					thenCalledOrThrow = true;
					return reject(r)
				});
			} else { // 2.3.3.4
				return resolve(x)
			}
		} catch (e) { // 2.3.3.2
			if (thenCalledOrThrow) return;
			thenCalledOrThrow = true;
			return reject(e)
		}
	} else {
		resolve(x)
	} 
}
```

### Conclusion

The Promise/A++ standard is so delegated, implementation turns out to be enjoyable journey. 



### Reference

[^1]: Promises/A+ 2018, **Promises/A+**, https://promisesaplus.com