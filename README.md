Mocha Redux Integration
=======================
Integration testing of redux stores with mocha.

-----------------------------

**This project is no longer maintained.** It was originally concieved as a proof of concept of a redux module testing. We use the concept -- redux store testing independent on the state shape -- on several projects, however, maintenance of its Mocha API is too time consuming for its benefits, which can be achieved with the very simple [`MockStore`](src/MockStore.js).

-----------------------------

* [Motivation](#motivation)
* [Installation](#installation)
* [Usage](#usage)
* [API](#api)

Motivation
----------
As all components which operate over redux store (action creators, reducers and selectors) are pure functions, we can easily unit test them. However, such tests have two deficiencies:
1. They do not check that components work with each other. When a programmer modifies reducer and hence the state shape but neglects to change selectors accordingly, the store breaks down without unit tests detecting it.
2. They are fragile. When a programmer changes the state shape (e.g. for optimization), all reducer tests have to be rewritten despite the store function staying the same.

To solve these problems, we should test the whole structure, starting with action creators and finishing with selectors. Also, to allow state shape changes, we should only test input/output behaviour, that is:
1. Given a store's initial shape.
2. We apply a sequence of actions.
3. And test that selectors return expected values.

The aim of this project is to offer a [custom mocha UI](https://github.com/mochajs/mocha/wiki/Third-party-UIs) to facilitate these tests. The test structure seems to be best represented by given-when-then syntax with **given** being initial state, **when** applied actions and **then** assertions on selector results.

Note: This approach to testing was designed to test redux modules (see [Jack Hsu's article](https://jaysoo.ca/2016/02/28/organizing-redux-application/)).

Installation
------------
Mocha Redux Integration is built and tested on Mocha 4, however, it should probably run on Mocha 3 too.

```
npm install --save-dev mocha-redux-integration
```

Usage
-----

Mocha Redux Integration provides a mocha UI, which is loaded with the `--ui` option:

```
mocha --ui mocha-redux-integration [TEST FILES]
```

### Writing Tests

Imagine you want to test a reducer and a set of actions and selectors.

First, all your tests need to be contained inside a `feature`:

```javascript
feature('Character Search Module', characterReducer, () => {
    /* TESTS * /
});
```

When your selectors expect, that the reducer is placed in a particular place in the reducer tree (using `combineReducers`),
you can specify the path (note, it assumes you use `redux-immutable`):

```javascript
feature('Character Search Module', characterReducer, ['search', 'characters'], () => {
    /* TESTS */
});
```

Then you can build scenarios, each consisting of an initial state -- **given** -- a sequence of applied actions -- **when** -- and an assortment of assertions -- **then**:
```javascript
scenario('First query', () => {
    given(); // initial state
    when('search is initialized', setQuery, 'Bonehunters');
    then('there are no characters', getCharacters, (result) => result.should.be.empty());
});
```

Note that actions are specified as the action creator function and its arguments, separately. This is necessary to catch errors in action creators.
Similarly, selector functions and its arguments are specified separately. The assertion is in the form of a callback.

You can also store a resulting state of a scenario and use it as an inital state of another one:
```javascript
scenario('First query', () => {
    /* ... */
    result('initialized search'); // you can also use string constant
});
scenario('Adding characters', () => {
    given('initialized search');
    /* ... */
});
```

For more information, you can go through the [example module](example) or read the [API](#api). 

### ESLint

As a Mocha UI this library defines several global variables, which have to be specified in order for ESLint to recognize them.
This is done by adding the following section to your `.eslintrc.json` (or adding them to existing globals):
```
    "globals": {
        "feature": false,
        "scenario": false,
        "given": false,
        "when": false,
        "then": false,
        "thenP": false,
        "result": false
    }
``` 

### File Names and Other Tests

I generally recommend putting redux integration tests into `redux.spec.js` files. This can, however, interfere with other tests,
which are commonly called using the `**/*.spec.js` glob. In this case, you can execute the other tests with a negative glob:
```
mocha **/!(redux).spec.js
```

Still, mocha redux integration includes mocha's BDD interface, thus making it possible to run integration tests along
with all the other tests. You might even use BDD features, such using as `describe` to wrap integration tests. 

API
---

* [`MockStore(reducer, path?)`](doc/MockStore.md)
* [Mocha UI](doc/MochaUI.md)
    * [`feature(title, reducer, path?, fn)`](doc/MochaUI.md#featuretitle-reducer-path-fn)
    * [`scenario(title, fn)`](doc/MochaUI.md#scenariotitle-fn)
    * [`given(title?)`](doc/MochaUI.md#giventitle)
    * [`when(title, actionCreator, ...params)`](doc/MochaUI.md#whentitle-actioncreator-params)
    * [`when(title, [actionCreator, ...params], [actionCreator, ...params], ...)`](doc/MochaUI.md#whentitle-actioncreator-params-actioncreator-params-)
    * [`then(title, selector, ...params, assertion)`](doc/MochaUI.md#thentitle-selector-params-assertion)
    * [`thenP(title, selector, [...params1], [...params2], ..., assertion)`](doc/MochaUI.md#thenptitle-selector-params1-params2--assertion)
    * [`result(title)`](doc/MochaUI.md#resulttitle)
