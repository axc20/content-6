## booleans

Boolean variables and functions that return boolean values should be named with one of these prefixes: `is`, `has`, `can`, `should`, `did`

Examples: `isManagedAccount`, `hasChanges`, `shouldUpdate`

## functions

A well-named function describes what it will do. It should directly or indirectly answer:

- What is the **action** being taken?
- What is the **contenxt** being acted on?
- What type of data will be returned?

Examples: `getOrderType`, `isBuyOrder`, `handleBuyClick`

| action (verb)    | usage case                                                                                                          |
| ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| apply            | updates multiple values in a target object                                                                          |
| get              | returns some value immediately                                                                                      |
| set              | assigns a value to some variable or object property                                                                 |
| reset            | restores a variable or object to its initial value or state                                                         |
| fetch            | permanently eliminates a value, often via an API call                                                               |
| handle           | takes action in response to an event                                                                                |
| render           | returns a `React` component                                                                                         |
| format           | return an input in some predefined format, typically a string                                                       |
| sanitize         | "cleaning" the input by removing unwanted characters                                                                |
| transform        | converts some data source to a different structure                                                                  |
| normalize        | takes some value and converts it to a common format or value                                                        |
| create           | returns a new instance of some data structure                                                                       |
| init, initialize | provides an initial or defaultvalue to a variable or state to an object                                             |
| strip            | removes some component from a string                                                                                |
| convert          | change a value from one type to another                                                                             |
| sort             | sorts an array                                                                                                      |
| find             | returns an item from a list matching some criteria; when returning index, index should be used as secondary context |
| calculate        | applies a formula to the inputs and returns the result                                                              |

## import organization

Organizing import statements in a clean and consistent manner makes it easier to understand the dependencies required by your component.

### custom styles

```js
import styles from './MyComponent.scss';
```

### external dependencies

```js
// react
import React, { useState } from 'react';
import PropTypes from 'prop-types';

// other external dependencies (alphabetical ordering)
import classnames from 'classnames';
import formatNumber from 'formatNumber';
import typeChecker from 'typeChecker';
```

### components

```js
// project shared components
import Footer from 'components/common/Footer';
import Header from 'components/common/Header';

// child components
import Footer from './Footer';
import Header from './Header';
```

### data layer interactions

```js
import { fetchQuoteDetails } from 'data/api/quoteInfo';
import { transformQuoteDetail } from 'data/transforms';
```

### hooks

```js
// project shared custom hooks
import useFetch from 'hooks/useFetch';
import useObjectState from 'hooks/useObjectState';

// component custom hooks
import useFoo from './hooks/useFoo';
import useBar from './hooks/useBar';
```

### utilities

```js
import { formatDateEST } from 'utils/formatters';
import translate from 'utils/translate';
```

### constants

```js
import {
  AccountType,
  AccountTypeDescription,
  AccountStatus,
  AccountStatusDescription
} from 'constants/account';
import { DEFAULT_DATE_FORMAT } from 'constants/application';
```

## Model-View-ViewModel

MVVM is a design pattern used for user interfaces that allows for a separation of concerns. Models handle data access and business logic as well as application state for that business logic. Views display data to the user and handle events. ViewModels present data from the model to the view. In our applications, they also store view state and handle view logic.

When deciding whether MVVM is right for your project, consider the project's size. For smaller projects, a well structured component hierarchy may work well. Or you may want to put business logic into reusable hooks. For large projects and large teams, MVVM has been time tested and proven to work well.

### implementation

At \[company\], our MVVM React applications are typically built using MobX, which allows us to bind ViewModels to Views using observable properties.

#### applicationModel

This is a singleton model. Because it is exported as a singleton, we reference it with lowercase. It is created at startup and stores global application state. It also provides a reference to any shared models.

```js
// applicationModel.js
class ApplicationModel extends BaseModel {
  static create() {
    return new ApplicationModel();
  }
}

export default ApplicationModel.create(); // export a single instance
```

#### Application

This is a React component that serves as the Parent view for the application

```js
// Application.js
class Application extends React.Component {
  static propTypes = {
    viewModel: PropTypes.object
  };

  handleChange = nextState => {
    this.props.viewModel.setStateProperty(nextState);
  };

  render() {
    const { stateProperty } = this.props.viewModel;

    return <SomeComponent prop={stateProperty} onChange={this.handleChange} />;
  }
}
```

#### ApplicationViewModel

```js
// ApplicationViewModel.js
import applicationModel from 'data/models/applicationModel';

class ApplicationViewModel extends BaseViewModel {
  static create() {
    const models = { applicationModel };

    return new ApplicationViewModel(models);
  }

  constructor(models) {
    super();

    Object.assign(this, models);
  }

  // @action?
  setStateProperty(nextState) {
    this.stateProperty = nextState;
  }
}

export default ApplicationViewModel;
```

#### ApplicationProvider

This creates the `ApplicationViewModel` and provides it as a prop to the `Application` component. `*Provider` components should not contain additional logic.

```js
// ApplicationProvider.js
class ApplicationProvider extends React.Component {
  constructor(props) {
    super(props);

    this.viewModel = ApplicationViewModel.create();
  }

  render() {
    return <Application viewModel={this.viewModel} {...this.props} />;
  }
}
```

We also add an `index.js` for the `ApplicationProvider` so that users of this component can import `Application` rather than `ApplicationProvider`:

```js
// Application/index.js
export default from './ApplicationProvider';
```

#### Dependency Injection

The `static create()` method of `ApplicationViewModel` exists for dependency injection during unit testing because you probably don't want to use the `applicationModel` singleton when testing your view models since it cannot be reset properly.

```js
describe('ApplicationViewModel', () => {
  it('should return correct value from applicationModel', () => {
    const mockApplicationModel = { stateProperty: true };
    const sut = ApplicationModel.create({
      applicationModel: mockApplicationModel
    });

    expect(sut.computedStateProperty).toEqual(true);
  });
});
```

### separation of concerns

The MVVM pattern allows for a separation of concerns between business logic and view logic.

#### views should only interact with ViewModels

```js
// applicationModel.js
class Application extends React.Component {
  render() {
    return this.props.viewModel.accountsByNickname.map(account => (
      <Account account={account} />
    ));
  }
}
```

```js
// ApplicationViewModel.js - add getter
class ApplicationModel extends BaseModel {
  get accountsByNickname() {
    return this.applicationModel.accountsByNickname;
  }
}
```

#### models handle data access via services

One of the roles of models is to provide access to APIs (i.e., HTTP services). In below example, `quotesOptionService` provides methods that make HTTP requests via a library like `axios` or the `fetch` API. A model can interact with more than one API, but in general, the model and these APIs/services should be constrained to a single business domain such as accounts or positions, or in the case of the example below, options.

```js
// OptionChainModel.js
class OptionChainModel extends BaseModel {
  static create() {
    const services = { quotesOptionService };

    return new OptionChainModel(services);
  }

  constructor(services) {
    super();

    Object.assign(this, services);
  }

  fetchOptionChain(symbol) {
    this.set('busy', true);

    this.quotesOptionService
      .fetchOptionChainSymbol(symbol)
      .then(options => this.set('options', options))
      .catch(err =>
        this.logger.error('OptionChainModel.fetchOptionChain error', error)
      );
  }
}
```

Models may also transform and/or aggregate data from the service into a format more usable by the frontend. Transform logic can also exist in the service file.

## style guide

### arrow functions

When using anonymous functions such as inline callback, use an arrow function as it creates a version of the function that executes in the context of `this` and is a more concise syntax. If the function is more complicated with a lot of logic, it may make sense to use a named function expression isntead.

### classes and constructors

Always use `class`. Avoid manipulating `prototype` directly.

```js
// bad
function Queue(contents = []) {
  this.queue = [...contents];
}

Queue.prototype.pop = function () {
  const value = this.queue[0];
  this.queue.splice(0, 1);
  return value;
};

// good
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }

  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}
```

Use `extends` for inheritance. This is a built-in way to inherit prototype functionality without breaking `instanceof`.

```js
class PeekableQueue extends Queue {
  peek() {
    return this.queue[0];
  }
}
```

Consider returning `this` from methods to help with method chaining.

```js
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }

  setHeight(height) {
    this.height = height;
    return this;
  }
}

const luke = new Jedi();

luke.jump().setHeight(20);
```

Classes have a default constructor if one is not specified. An empty constructor function or one that just delegates to a parent class is unnecessary.

```js
// bad
class Jedi {
  constructor() {}

  getName() {
    return this.name;
  }
}

class Rey extends Jedi {
  constructor(...args) {
    super(...args);
  }
}

// good
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
    this.name = 'Rey';
  }
}
```

### comments

```js
/**
 * make() returns a new element based on passed in tag name
 *
 * @param {String} tag
 * @return {Element} element
 */
function make(tag) {
  // ...
  return element;
}

// prefixing comments with FIXME or TODO helps other developers
// different from regular comments because they are actionable
// FIXME: shouldn;t use a global here
// TODO: total should be configurable by an options param
```

### comparison operators & equality

- **Objects** evaluate to `true`
- **Undefined**, **Null** evaluates to `false`
- **Booleans** evaluate to value of the boolean
- **Numbers** evaluate to `false` if `+0`, `-0`, or `NaN`, otherwise `true`
- **Strings** evaluate to `false` if an empty string `''`, otherwise `true`

Use braces to create blocks in `case` and `default` clauses that contain lexical declarations (e.g. `let`, `const`, `function`, `class`). Why? Lexical declarations are visible in the entire `switch` block but only get initialized when assigned, which only happens when its `case` is reached. This causes problems when multiple `case` clauses attempt to define the same thing.

```js
// bad
switch (foo) {
  case 1:
    let x = 1;
    break;
  case 2:
    const y = 2;
    break;
  case 3:
    function f() {}
    break;
  default:
    class C {}
}

// good
switch (foo) {
  case 1: {
    let x = 1;
    break;
  }
  case 2: {
    const y = 2;
    break;
  }
  case 3: {
    function f() {}
    break;
  }
  default: {
    class C {}
  }
}
```

### control statements

```js
// don't use selection operators in place of control statements
// bad
!isRunning && startRunning();

// good
if (!isRunning) {
  startRunning();
}
```

### events

When attaching data payloads to events (DOM events), pass an object literal (also known as a "hash") instead of a raw value. This allows a subsequent contributor to add more data to the event payload without finding and updating every handler for the event.

```js
// bad
const event = new CustomEvent('foo-happens', fooId);
el.dispatchEvent(event);
document.addEventListener('foo-happens', (e, fooId) => {});

// good
const event = new CustomEvent('foo-happens', { data: { fooId } });
el.dispatchEvent(event);
document.addEventListener('foo-happens', (e, data) => {
  // do something with data.fooId
});
```

### functions

```js
// never name a paramater arguments, as this will take precedence over
// the arguments object that is given to every function scope
// bad
function foo(name, options, arguments) {}

// good
function foo(name, options, args) {}

// never use arguments; opt to use rest syntax ...
// rest is explicit about which arguments you want pulled
// rest arguments are a real Array and not merely Array-like like arguments
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}

// prefer the use of spread operator ... to call variadic functions
// bad
const x = [1, 2, 3, 4, 5];
console.log.apply(console, x);

// good
const x = [1, 2, 3, 4, 5];
console.log(...x);
```

### hoisting

`var` declarations get hoisted to the top of their closest enclosing function scope, but their assignment does not. `const` and `let` declarations are blessed with a new concept called Temporal Dead Zones.

```js
// assuming no notDefined global variable, this would not work
function example() {
  console.log(notDefined); // => throws a ReferenceError
}

// creating a variable declaration after referencing the variable will work
// due to variable hoisting; note the assignment value of `true` is not hoisted
function example() {
  console.log(declaredButNotAssigned); // => undefined
  var declaredButNotAssigned = true;
}

// can be rewritten as:
function example() {
  let declaredButNotAssigned;
  console.log(declaredButNotAssigned); // => undefined
  declaredButNotAssigned = true;
}

// using const and let
function example() {
  console.log(declaredButNotAssigned); // => throws a ReferenceError
  console.log(typeof declaredButNotAssigned); // => throws a ReferenceError
  const declaredButNotAssigned = true;
}
```

Anonymous function expressions hoist their variable name, but not function assignment.

```js
function example() {
  console.log(anonymous); // => undefined
  anonymous(); // => TypeError anonymous is not a function
  var anonymous = function () {
    console.log('anonymous function expression');
  };
}
```

Named function expressions hoist the variable name, not the function name or the function body.

```js
function example() {
  console.log(named); // => undefined
  named(); // => TypeError named is not a function
  superPower(); // => ReferenceError superPower is not defined
  var named = function superPower() {
    console.log('Flying');
  };
}

// same is true when function name is same as variable name
function example() {
  console.log(named); // => undefined
  named(); // => TypeError named is not a function
  var named = function named() {
    console.log('named');
  };
}
```

Function declarations hoist their name and the function body.

```js
function example() {
  superPower(); // => Flying

  function superPower() {
    console.log('Flying');
  }
}
```

### iterators

Don't use iterators. Prefer JS higher-order functions instead of loops like `for-in` or `for-of`. Why? This enforces immutability, as dealing with pure functions that return values is easier to reason about than side effects.

Use `map()`, `every()`, `filter()`, `find()`, `findIndex()`, `reduce()`, `some()` to iterate over arrays and `Object.keys()`, `Object.values()`, `Object.entries()` to produce arrays so you can iterate over objects.

### naming conventions

Do not use trailing or leading underscores. JS does not have concept of privacy in terms of properties or methods. Although a leading underscore is a common convention to mean "private", in fact, these properties are fully public and are part of your public API contract. If you want something to be "private", it must not be observably present.

Don't save references to `this`. Use arrow functions or `bind`.

```js
// bad
function foo() {
  const self = this;

  return function () {
    console.log(self);
  };
}

// good
function foo() {
  return () => {
    console.log(this);
  };
}
```

Acronyms and initialisms should always be capitalied, or all lowercased.

```js
const HTTPRequests = []; // good
const httpRequests = []; // also good
const requests = []; // best
```

### objects

```js
// use object method shorthand
// bad
const atom = {
  value: 1,
  addValue: function (value) {
    return atom.value + value;
  }
};

// good
const atom = {
  value: 1,
  addValue(value) {
    return atom.value + value;
  }
};

// prefer object spread operator to shallow-copy objects
// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 });
delete copy.a; // bad, mutates original

// good
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 };
const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
```

### references

```js
// note that both let and const are block-scoped
{
  let a = 1;
  const b = 1;
}
console.log(a); // ReferenceError
console.log(b); // ReferenceError
```

### standard library

The [Standard Library](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects) contains utilities that are functionally broken out but remain for legacy reasons.

```js
// use Number.isNaN instead of global isNaN
// bad
isNaN('1.2.3'); // true

// good
Number.isNaN('1.2.3'); // false
Number.isNaN(Number('1.2.3')); // true

// use Number.isFinite instead of global isFinite
// bad
isFinite('2e3'); // true

// good
Number.isFinite('2e3'); // false
Number.isFinite(parseInt('2e3', 10)); // true
```

### type casting & coercion

```js
// => this.reviewScore = 9;
const totalScore = String(this.reviewScore);

// Numbers: use Number for type casting and parseInt always with a radix for parsing strings
const inputValue = '4';
const val = Number(inputValue);
const val = parseInt(inputValue, 10);

// Booleans
const age = 0;
const hasAge = !!age; // good
const hasAge = Boolean(age); // best
```

If for whatever reason you are doing something wild and `parseInt` is your bottleneck and need to use bitshift for performance reasons, leave a comment explaining why and what you're doing.

```js
/**
 * parseInt was the reason my code was slow
 * Bitshifting the String to coerce it to a Number made it a lot faster
 */
const val = inputValue >> 0;
```

Be careful when using bitshift operations. Numbers are represented as 64-bit values, but bitshift operations always return a 32-bit integer. Bitshift can lead to unexpected behavior for integer values larger than 32 bits. Largest signed 32-bit Int is 2,147,483,647.

```js
2147483647 >> 0; // => 2147483647
2147483648 >> 0; // => -2147483648
2147483649 >> 0; // => -2147483647
```
