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

---

COLLAPSe

## Standard built-in objects

### Date

```javascript
const date = new Date(Date.UTC(2021, 11, 20, 3, 0, 0));
date.toLocaleString('en-US'); // → "12/19/2012, 10:00:00 PM"
// Date.now();
// new Date('1/7/2021') // .getFillYear(), .getMonth(), .getDate()
```

### Intl

```javascript
const date = new Date(Date.UTC(2020, 11, 20, 3, 23, 16, 738));
new Intl.DateTimeFormat('en-US').format(date); // → "12/19/2020"
new Intl.DateTimeFormat('en-US', { dateStyle: 'full' }).format(date); // → "Saturday, December 19, 2020"
// Intl.NumberFormat
```
