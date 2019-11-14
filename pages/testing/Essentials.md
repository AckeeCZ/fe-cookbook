# Testing Essentials

For testing generally, you will always need at least **testing framework** & usually also **mocking library**.

We have a luck that our testing framework also includes mocking funcionality (& much more, later about it) so that's all we need for now. There is already a recipe about [**how to start with Jest**](/pages/libraries/Jest.md) - our testing framework. Take all the text here as and addition to it.

## TOC

* [Matchers](#matchers)
  * [Comparation helpers](#comparation-helpers)
* [Mocking](#mocking)
  * [Basic mocks](#basic-mocks)
  * [Timer mocks](#timer-mocks)

---

## Matchers

> Don't worry about mocking sometimes used in examples, we'll discuss it in next section 😉

What is it? Long story short: "function that test expected value against real one". They make subject of testing more obvious.

There is [big bunch of matchers](https://jestjs.io/docs/en/expect#methods), but the ones most used in tests are `toBe`, `toEqual`, `toHaveBeenCalledTimes` and `toHaveBeenCalledWith`. Let's review theme one by one and look at their usage and props/cons.

### `.toBe(value)`

* Simple, short and easy to write
* Convenient for comparation of primitive types
* Compares values with `Object.is` which is similar to `===`

```js
test('game is done', () => {
    const game = { done: true }
    
    expect(game.done).toBe(true) // passes
    expect(game).toBe({ done: true }) // fails
})
```

### `.toEqual(value)`

* Does deep compare so it's useful for comparation complex objects
* When comparing two errors, only `message` property is used for comparation

```js
test('game is done', () => {
    const game = { done: true }
    
    expect(game).toEqual({ done: true }) // passes
})
```

Most of other helpers (eg. `.toHaveLength` matcher) are just syntax sugar that could be rewritten to `.toEqual` usage.

```js
test('there are exactly two players', () => {
    const players = [
        { id: 1, name: 'Novak Djokovic' },
        { id: 2, name: 'Rafael Nadal' },
    ]
    
    expect(players).toHaveLength(2) // passes
    expect(players.length).toEqual(2) // also passes
})
```

### `.toHaveBeenCalledTimes(count)`

* Test if mock function was called n-times exactly
* Doesn't care about call arguments

```jsx
import firebase from './myFirebase'
const getFirebaseUser = firebase.auth().currentUser

test('get user from firebase', () => {
    jest.mock('./myFirebase', () => ({
         auth: jest.fn().mockReturnValue('my user'),
    }))
     
    const user = getFirebaseUser()

    expect(firebase.auth).toHaveBeenCalledTimes(1)
})
```

### `.toHaveBeenCalledWith(arg1, arg2, ...)`

* Test what arguments was function called with
* Sometimes it's useful to combine it with `.toHaveBeenCalledTimes` to check that we're comparing right call arguments

```jsx
import firebase from './myFirebase'
const loginWithToken = token => {
    const credential = firebase.auth.GoogleAuthProvider.credential(token)
    firebase.auth.signInWithCredential(credential)
}

test('log user in with google credential', () => {
    jest.mock('./myFirebase', () => ({
         auth: {
             GoogleAuthProvider: {
                 credential: jest.fn().mockReturnValue('fake credential'),
             },
             signInWithCredential: jest.fn()
        }
    }))
    const token = 'fake token'
     
    loginWithToken(token)

    expect(firebase.auth.GoogleAuthProvider.credential).toHaveBeenCalledWith(token)
    expect(firebase.auth.signInWithCredential).toHaveBeenCalledWith('fake credential')
})
```

### Comparation helpers

There some comparation helpers that easy your work when comparing complex objects, or want to check only particular arguments of a function call.

Most of the helpers is negatable (usually those which makes sense to negate) by prepending `.not`, eg. `expect.not.arrayContaining`.

#### `expect.anything()` & `expect.any(constructor)`

Useful if you wanna test a function was called with argument bude you don't care about exact value. The `.any` variant allows you to check a type of the argument.

```js
const mapUsers = (users, mapper) => {
    users.map(user => mapper(user))
}

test('map users with custom mapper', () => {
    const mapper = jest.fn()
    const users = [ { id: 1, name: 'John'}, { id: 2, name: 'Jack'} ]

    mapUsers(users, mapper)

    expect(mapper).toHaveBeenCalledWith(expect.anything())
})
```

```js
const mapUserNamesToRandomIds = (names, mapper) => {
    names.map(name => mapper(`${name}-${Math.random()}`))
}

test('map user names with custom mapper and provide a random id', () => {
    const mapper = jest.fn()
    const users = [ 'John', 'Jack' ]

    mapUsers(users, mapper)

    expect(mapper).toHaveBeenCalledWith(expect.any(String))
})
```

#### `expect.arrayContaining(array)` & `expect.objectContaining(object)`

Very useful when you don't want to check whole array/object, but only particular items/properties that are just important for you.

```js
const values = [1, 2, 3, 4, 5, 6]

test('Each attempt contain all values', () => {
    const attempts = [
        [4, 1, 6, 7, 3, 5, 2, 5, 4, 6],
        [8, 6, 3, 5, 1, 2, 4, 0, 7],
    ]
    expect(attempts[0]).toEqual(expect.arrayContaining(values))
    expect(attempts[1]).toEqual(expect.arrayContaining(values))
})
```

```js
const iterator = 0
const generateRandomPoint = () => ({
    name: `point${iterator++}`,
    type: 'point',
    x: Math.round(Math.random() * 1000),
    y: Math.round(Math.random() * 1000),
})

test('generated point has correct type', () => {
    const point = generateRandomPoint()

    expect(point).toEqual(expect.objectContaining({
        type: 'point',
    }))
})
```

It gets more poweful when you start to combine them with other mentioned helpers

```js
test('random coordinates are generated for point', () => {
    const point = generateRandomPoint()

    expect(point).toEqual(expect.objectContaining({
        x: expect.any(Number),
        y: expect.any(Number),
    }))
})
```

#### `expect.stringContaining(string)` & `expect.stringMatching(string|regexp)`

You wouldn't use these helpers for simple value comparation, cause there are `toContain` and `toMatch` matchers, that are easier and more suitable to use. But you will find them handy in combination with `.arrayCotaining` and `.objectContaining`

```js
test('unique name is generated for point', () => {
    const point1 = generateRandomPoint()
    const point2 = generateRandomPoint()

    expect(point1.name).not.toEqual(point2)
    expect(point).toEqual(expect.objectContaining({
        name: expect.stringContaining('point'),
    }))

    expect(point).toEqual(expect.objectContaining({
        name: expect.stringMatching(/point\d{1}/),
    }))
})
```

---

That's  all about matchers for now, don't forget to discover all of them at the [documentation page](https://jestjs.io/docs/en/expect#methods). There are plenty of other matchers that you might find useful like `.toContain`, `.toBeNull`, `.toThrow`, `.toMatch`, `toHaveLength`, etc. but the described ones are basics.

And one more thing - you can extend basic set of matchers by custom ones when you need some, eg. for some specific environments like [DOM](https://github.com/testing-library/jest-dom) or [React Native](https://github.com/testing-library/jest-native).

## Mocking

>Technique that allows you to focus on testing one specific part of code, faking other parts assuming they work as intended.

In case you din't do it yet, read more about some Jest basics to be ready for the examples:

* [how to return a value from mock](https://jestjs.io/docs/en/mock-functions#mock-return-values)
* [advanced mock implementation](https://jestjs.io/docs/en/mock-functions#mock-implementations)
* [how to test asynchronous](https://jestjs.io/docs/en/tutorial-async) code using
    * [callbacks](https://jestjs.io/docs/en/asynchronous#callbacks)
    * [promises](https://jestjs.io/docs/en/asynchronous#promises)
    * [async/await](https://jestjs.io/docs/en/asynchronous#async-await)


### Why we need mocking?

* **SPEED**‼ Less real code to execute means faster tests run.
* **Focus** When dependencies are mocked and only tested code is run, we can quickly identify why test fails.
* Of course, there is more reasons for mocking but I am not able to list all of them here. You can find more of them in Kent C. Doods [article about javascript mocks](#resources)

### Basic mocks

We will ilustrate it on mocking 3rd party library, but you can use it same way also for other sources, modules, etc. in app.

So there is a selector that uses `routingSelector` from `@ackee/chris` package and since there is no browser and no real location, we need to mock it to return some.


```js
// mySelectors.js
import { createSelector } from 'reselect'
import { routingSelector } from '@ackee/chris'

export const selectMenuItems = createSelector(
    routingSelector,
    location => {
        const currentPathname = location.pathname

        return ['home', 'about', 'team', 'purchase'].map(id => {
            const route = `/${id}`
            
            return {
                id,
                route,
                active: route === currentPathname,
            }
        })
    }
)
```

Notice:
* how we mimic location object and provide only properties we need
* how we mock a module and then import it to be able to "program" intended behaviour
* `selectMenuItems` can receive fake state object `{}` since routingSelector is mocked and doesn't use it

```jsx
// __tests__/mySelectors.js
import { selectMenuItems } from '../mySelectors.js'
import { routingSelector } from '@ackee/chris'

jest.mock('@ackee/chris')

describe('selectMenuItems', () => {
    it('returns all menu items', () => {
        routingSelector.mock.returnValue({ pathname: '/' })
        
        const items = selectMenuItems({})
        
        expect(items).toHaveLength(4)
    })
    
    it('marks current route active', () => {
        routingSelector.mock.returnValue({ pathname: '/team' })

        const items = selectMenuItems({})
        
        expect(items[2]).toHaveProperty('active', true)
    })
    
    it('does not mark other than current route as active', () => {
        routingSelector.mock.returnValue({ pathname: '/team' })
        
        const items = selectMenuItems({})
        
        expect(items[0]).toHaveProperty('active', false)
        expect(items[1]).toHaveProperty('active', false)
        expect(items[3]).toHaveProperty('active', false)
    })
})
```

in the example above whole `@ackee/chris` module is mocked. Sometimes you need only a certain part of a module to be mocked and the rest remain untouched. That's called **partial mock** and it's easy to accomplish it.

In the previous example we would just change the mock command and in following example, `routingSelector` will be mocked function whereas `combinedDependenciesHandlers` will be a real function.

```js
import { routingSelector, combineDependenciesHandlers } from '@ackee/chris'

jest.mock('@ackee/chris', () => {
    return {
        ...jest.requireActual('@ackee/chris'),
        routingSelector: jest.fn(),
    }
})
```

You can also check [module examples in jest repository](https://github.com/facebook/jest/tree/master/examples/module-mock) as well as [manual mock examples](https://github.com/facebook/jest/tree/master/examples/manual-mocks).

### Timer mocks

Sometimes you code depends on some timing, maybe you use `setTimeout` or other kind of delaying code by time, and you would like to conrol the timing for tests. 

That easily solved in Jest by faking timers with [`jest.useFakeTimers`](https://jestjs.io/docs/en/timer-mocks).

```js
function delayFunction(callback, delayInSeconds = 1) {
    setTimeout(() => {
        callback()
    }, delayInSeconds * 1000)
}
```

Notice:

* usage of `advanceTimersByTime`
* how value of time is cumulated

```js
jest.useFakeTimers()

test('delay function call by 1 second if no delay not defined', () => {
    const callback = jest.fn()
    
    delayFunction(callback)

    expect(callback).not.toHaveBeenCalled() // 0 ms elapsed since delayFunction call

    jest.advanceTimersByTime(1000) 

    expect(callback).toHaveBeenCalled()  // 1000 ms elapsed since delayFunction call
    expect(callback).toHaveBeenCalledTimes(1)
})


test('delay function call by custom delay in seconds', () => {
    const callback = jest.fn()
    
    delayFunction(callback, 1.5)

    expect(callback).not.toHaveBeenCalled()  // 0 ms elapsed since delayFunction call

    jest.advanceTimersByTime(1000)

    expect(callback).not.toHaveBeenCalled()  // 1000 ms elapsed since delayFunction call

    jest.advanceTimersByTime(500)

    expect(callback).toHaveBeenCalled()  // 1500 ms elapsed since delayFunction call
    expect(callback).toHaveBeenCalledTimes(1)
})
```

There are also other timers mocking helpers like [`runAllTimers`](https://jestjs.io/docs/en/timer-mocks#run-all-timers) or [`runPendingTimers`](https://jestjs.io/docs/en/timer-mocks#run-pending-timers). Be aware of them in case you would need them but I found `advanceTimersByTime` sufficient for most cases.

----

### Resources

* [What is javascript mock](https://kentcdodds.com/blog/but-really-what-is-a-javascript-mock) by Kent C. Doods