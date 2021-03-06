# React + Redux 
### Tips and Best Practices for Clean, Reliable & Maintainable Code

by Cody Barrus

![](construction.jpg)

---

# Clean Code

- Easy to understand
- Easy to change

---

# Code that isn't clean rots

Code rot is the slow deterioration of code over time that will eventually lead to code that is either tightly coupled, filled with technical debt, or difficult to maintain.

---

# React/Redux Data Flow
## *In 30 seconds!*

![](mirror.jpg)

---

![fit](reduxlifecycle.png)

---

### React/Redux Data Flow 30 Second Review

- Our redux code lives in a module
    + Called DUCKS
    + All related constants, actions/action creators, and the reducer are in a single file.


---

### React/Redux Data Flow 30 Second Review

- An action is  `dispatch()`ed
- Passes data to the reducer.
- The reducer updates the store.


---

### React/Redux Data Flow 30 Second Review

- The store sends new data via `connect()` to component props.
- `class` component calls render lifecycle, and updates the view.
- Stateless Functional Component (SFC) is called and returns updated view.


---

### React/Redux Data Flow 30 Second Review

- Component listens to events which `dispatch()` next action via `connect()`.
- Repeat.


---

# Redux Modules
## The data layer

![](cake.jpg)

---

#### Redux Modules

- Modules consist of
    + Constants
    + Actions
    + Reducer

---

#### Redux Modules

# Actions

![](fourwheels.jpg)

---

#### Redux Modules > Actions

- Actions are payloads of information that send data from your application to your store.
- They are the only source of information for the store.

---

#### Redux Modules > Actions

```javascript
// Constant 
const ADD_TODO = '@todos/ADD_TODO'

// Action
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

---

#### Redux Modules > Actions

- A simple action just passes data and a type to the reducer

```javascript
const addToDo = (toDo) => ({
    type: ADD_TODO,
    toDo    
});
```

---

#### Redux Modules > Actions

- Actions can take advantage of the `thunk` and `promise` middlewhere.
    - Adds flexibility to your Actions by giving them access to `state` and allowing them to return promises.
    - Can easily get out of hand.

---

#### Redux Modules > Actions

```javascript
import {getLists} from 'apiCalls';

// Arbitrary example
const complexAddToDo = (toDo) => {
    return (dispatch, getState) => {
        const {userPrefs} = getState().user;

        return getLists.then(list => {
            dispatch(populateLists(list))
        }).then(() => {
            if (userPrefs.isAwesome) {
                dispatch(addToDo(toDo))
            }
        })
    }
}
```

---

## Functions should only do one thing and do it well.

---

#### Redux Modules > Actions
## A few pointers to keep these actions reasonable:

+ Keep complexity out of your Actions.
+ Keep Actions pure (w/o side effects).
+ Prefer data manipulation in the reducer.

---

#### Redux Modules > Actions

- Keep API calls in their own util.
    + Keeps actions cleaner
    + Is simpler to unit test

---

#### Redux Modules > Actions
## getState

+ Don't call `getState` unecessarily.
+ Don't use `getState` for getting data that's handled by the local reducer.
    + Insead, dispatch an action and access data in the reducer.

---

#### Redux Modules > Actions
## getState continued

+ Call `getState` only once near the top of your function.
+ Always treat data from the store as though it were immutable.

---

#### Redux Modules > Actions

```javascript
const complexAddToDo = (toDo) => {
    return (dispatch, getState) => {
        const {
            user: {userPrefs},
            movies: {titles},
            ....
        } = getState();

        ....
    }
}
```

---

## Actions Summary

- Keep actions pure and simple.
- API calls live in a separate util.
- `thunk` and `promise` middleware add power, but with great power comes great responsibility.

---

#### Redux Modules
# Reducer

![](bird.jpg)

---

#### Redux Modules > Reducer

- The reducer handles state changes in response to an action.

---

#### Redux Modules > Reducer

```javascript
const CHANGE_THING = '@todoModule/CHANGE_THING';

const initalState = {
    thing: ''
}

export default const myReducer = (state = initialState, action = {}) => {
    switch (action.type) {
        case CHANGE_THING:
            return {
                ...state,
                thing: action.value
            }
        default:
            return {
                ...state
            };
    }
};
```

---

#### Redux Modules > Reducer

## Tips for clean reducers:

- The best reducers specialize in a single concern.
- Complex data manipulation lives in the reducer.

---

#### Redux Modules > Reducer

## More tips for clean reducers:

- Use helper functions and utils.
- Reducers can listen for actions from another module if needed.

---

#### Redux Modules > Reducer > Listen to other modules actions

```javascript
// expenseHomeModule.js
const RESET_EXPENSE_STATE = '@expenseHome/RESET_EXPENSE_STATE';

// expenseItemizationModule.js
import {RESET_EXPENSE_STATE} from '../expenseHomeModule';

export default const myReducer = (state = initialState, action = {}) => {
    switch (action.type) {
        ....
        case RESET_EXPENSE_STATE:
            return {
                ...initialState
            }
        ....
    }
}

```

---

#### Redux Modules
# Constants

- Preface the constant with the name of the reducer.
- **Ok:** `const SUBMIT_REPORT = 'SUBMIT_REPORT'`
- **Better:** `const SUBMIT_REPORT = '@report/SUBMIT_REPORT`

---

# Components

![](typewriter.jpg)

---

### Components
# Props

![](props.jpg)

---

#### Components > Props

## Passing Props

- There are several ways to pass props:
    + Component to Component
    + Connected Component
    + Higher Order Component

---

![](time-to-render.png)

---

### Passing Props
# Component to Component

![](wires.jpg)

---

#### Components > Props > Component to Component

- Simplest method of passing data.
- Parent component gives child data through props.

---

#### Components > Props > Component to Component

```javascript
// Home.jsx

class Home extends React.Component {
    ....
    render() {
        return (
            <div>
                <MyComponent
                    prop1={this.state.thing1}
                    prop2={this.props.thing2}
                    prop3={this.props.thing3}
                    ....
                />
            </div>
        )
    }
}
```

---

#### Components > Props > Component to Component

- **Good for:** 
    - A small number of props.
    - Parent component methods.
    - Data local to parent component.

---

#### Components > Props > Component to Component

- **Not good for:**
    + A large number of props.
    + Passing actions to child component.
    + Passing data from Redux store to child component.
    + Passing `{...props}`.

---

#### Components > Props > Component to Component

```javascript
// Home.jsx

class Home extends React.Component {
    ....
    render() {
        return (
            <div>
                <MyComponent
                    prop1={this.state.thing1} // OK
                    prop2={this.props.thing2} // NOT SO GOOD
                    prop3={this.props.thing3} 
                    ....
                />
            </div>
        )
    }
}
```

---

#### Components > Props > Component to Component

## Passing props from state

- **Good when...**
    + data is local to component AND
    + child component is presentational (dumb) component. *ie. open state of a modal*

---

#### Components > Props > Component to Component

## Passing props from state

- **Not good when...**
    + data is better handled in the redux reducer. *ie. data is required for multiple components*

---

#### Components > Props > Component to Component

## Don't pass parent props to child

+ Creates tight coupling between components.
+ Makes components difficult to maintain.
+ Simple data changes may force you to refactor at least 2, possibly more, components.
+ Instead, use connected pattern (more later).

---

#### Components > Props > Component to Component

```javascript
class Home extends React.Component {
    render() {
        return (
            <div>
                <MyComponent
                    {...this.props} // :(
                />
            </div>
        )
    }
}
```

---

#### Components > Props > Component to Component

## Don't Pass `{...this.props}`

+ Causes even tighter coupling involving at least 3 components!
    + Grandparent (where data is coming from).
    + Parent (where component is initialized)
    + Child (where data is being utilized).

*But wait, there's more!*

---

#### Components > Props > Component to Component

- More `props` = more time to render.
- Forms especially suffer from pref hit.
- Spreading props leads to difficult to read/reason about code.

---

#### Components > Props > Component to Component

## Caveat

- Higher Order Components often make use of `{...props}`. 
- **Just think about when this works well and when it doesn't.**

---

#### Components > Props > Component to Component

## Summery

- Explicitly list props.
- Avoid passing parent props to children (ie. `prop={this.props.foo}`). 
- Prefer the connected pattern.
- Avoid spreading props from one component to another (ie. `{...props}`).

---

### Passing Props
# Connected Component Pattern

![](light.jpg)

---

#### Components > Props > Connected Component

- A connected component uses the `react-redux` connect function to pass props directly from state.

---

#### Components > Props > Connected Component

- **The Good**:
    - Greately reduces the code complexity.
    - Removes tight coupling of components.
    - Acts as documentation on actions your components depend on.

---

#### Components > Props > Connected Component

- **The Bad**:
    - Requires a little more boilerplate code.

---

#### Components > Props > Connected Component

```javascript
// MyComponent.jsx
import {connect} from 'react-redux';

const MyComponent = ({ prop1, prop2, prop3 }) => {
    return (
        <div>
            {`I am a ${prop1} that ${prop2} when ${prop3}`}
            ....
        </div>
    )
}

const mapStateToProps = state => ({
    prop1: state.expense.prop1,
    prop2: state.itemization.prop2,
    prop3: state.user.prop3    
});

export defualt connect(mapStateToProps)(MyComponent);
```

---

#### Components > Props > Connected Component

```javascript

// Home.jsx

class Home extends React.Component {
    render() {
        return (
            <div>
                <MyComponent />  // :) State data is already mapped to props
            </div>
        )
    }
}
```

---

#### Components > Props > Connected Component

- Both components are independant of one another.
- It can be dropped anywhere and will always work as intended.

---

#### Components > Props > Connected Component

- Keeps data flow through your app direct and simple.
- No need to create a separate container file. That simply adds complexity without any real benefit.

---

#### Components > Props > Connected Component

- Remember: avoid passing unnecessary props.

```javascript
// avoid patterns like this, they'll cause a hit to your performance
// and obfuscate your dependencies

connect(state => ({
    ...state // Nope!
}));
```

---

#### Components > Props > Connected Component

- Instead, explicity require each prop needed for that particular component.

```javascript
connect(state => ({
    movieTitles: state.movies.titles,
    bookTitles: state.books.titles,
    tvShowTitles: state.tvShows.titles
}));
```

---

#### Components > Props > Connected Component

- Benefits of explicitly declaring props include:
    + Simple to read.
    + Easy to see when a component has expanded past its concern.
    + Only maps required props, decreasing time to render.

---

#### Components > Props > Connected Component
## Summary

+ Prefer Connected Pattern over Component to Component pattern.
+ Connected componets are simplier to maintain, and reduce tech debt significantly.
+ Don't spread state.
+ Be explicit.

---

### Passing Props
# Higher Order Components (HOC)

![fit](tall cliff.jpg)

---

#### Components > Props > Higher Order Components

- A function that takes a component and returns a new component.
- Good for reusing component logic.
- Make it easy to layer on behavior while maintaining a separation of concerns.

---

#### Components > Props > Higher Order Components

```javascript
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it.
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

---

#### Components > Props > Higher Order Components

- Adds additional functionality, or injects data, into the component it wraps.
- **Good for:**
    - Behaviour that is needed throughout the app.
    - Common data sets needed in several components.

---

#### Components > Props > Higher Order Components

- **Warning:** Excessive use of HOCs can hurt performance.
- If you manage your `props` well, the hit will be small.
- If not, your User Experience could deminish.

---

### Components
# Handling Data

![](handles.jpg)

---

# Where should I manipulate data?
 
- Component
- mapStateToProps
- Redux module

---

#### Components > Props > Handling Data

- Data is used to render the view:
    + Pass data through props
    + manipulate in the component

---

#### Components > Props > Handling Data

- Data is used to render the view but may change based on state:
    + Pass data through mapStateToProps
    + manipulate in the mapStateToProps function before passing to props

---

#### Components > Props > Handling Data

```javascript

const mapStateToProps = state => {
    return {
        showModal: state.thing.isAwesome && state.otherThing.isGreat
    }
}

```

---

#### Components > Props > Handling Data

- Data is used to update state based on user action
    + Dispatch action from component
    + Handle data manipulation in module

---

### Components

# Props Summery

![](chalkboard.jpg)

---

- Avoid passing unnecessary props.
- Connected Components **>** Higher Order Compontents **>** Component to Component.
- Break components requiring too many props into several, more focused components.
- All these rules have exceptions. Every circumstance is different.

---

### Components
# Class Components

![](class.jpg)

---

#### Components > Class Components

**The basic building block of every React app**

---

#### Components > Class Components

- **The good:**
    + Very powerful.
    + Have access to lifecycle methods and `this.state`.

---

#### Components > Class Components

- **The bad:**
    + Can easily become over complicated, too big, or difficult to maintain.
    + `this.state` is the source of many bugs. Better to handle data in the Redux module or through props in most cases.

---

### Components
# Stateless Functional Components (SFCs)

![](desk.jpg)

---

#### Components > Stateless Functional Components

- The simplest way to declare components.
- They are basic JavaScript functions that take props and return jsx.

---

#### Components > Stateless Functional Components

```javascript
const Welcome = (props) => <h1>Hello, {props.name}</h1>;

const Fairwell = ({prop1, prop2}) => {
    const calculateThings = (prop1, prop2) => {
        return prop1 + prop2;
    }

    return (
        <div>{calculateThings()}</div>
    )
}
```

---

#### Components > Stateless Functional Components

- **The good:**
    + Simpler & easier to maintain than `class` components.
    + Givin the same input, an SFC will always have the same output. *Not so with a `class` component*
    + Do not have access to `state` -- yes, that's a good thing ;)

---

#### Components > Stateless Functional Components

- **The bad**
    + SFCs do not have access to state or any React lifecycle methods.
    + That's it really...

---

#### Components > Stateless Functional Components

- SFCs **>** `class` components.
- `class` components are best used as the root component of a view, or for components that rely on lifecycle methods.
- *In all other cases, use SFCs.*

---

### Components
# Refs

![](arrows.jpg)

---

#### Components > Refs

- `ref`s are generally references to DOM elements within a component.
- There are two ways for a parent component to reach into a child component
    -  surfacing values or methods (such as event hanlders) through props.
    -  refs.

---

#### Components > Refs

```javascript
// with refs
componentDidMount() {
    this.refs.someWidget.focus()
}

// without refs
render() {
    return <Widget focused={true} //>;
}
```

---

#### Components > Refs

- **The good:**
    + Occasionally helpful.
    + Occasionally...

---

#### Components > Refs

- **The bad**
    + Increase function calls and property merging.
    + Can obscure a component's dependencies.
    + Can easily lead to tight coupling and debugging nightmares.

---

#### Components > Refs

# Summery

- Favor surfacing values or methods through props over `ref`s.

---

### Components
# State

![](bag.jpg)

---

#### Components > State

- There are several ways to handle the component state.
- `class` components have access to `this.state` whereas SFCs do not.
- Accessing and updating a components state is relatively painless.

---

#### Components > State

```javascript
class MyComponent extends React.Component {

    handleChange(value) {
        this.setState({
            foo: value   
        });
    }

    render (
        <span>{state.value}</span>
    )
}
```

---

#### Components > State

- **The good:**
    + Very easy.
    + Great for managing things that aren't related to data in the redux store. *ie. active states, is modal open, etc*

---

#### Components > State

- **The bad:**
    + Relying on component state too much can make components difficult to re-use and maintain.
    + As components multiply, frequent state manipulation can add to your technical debt.

---

#### Components > State

- **The bad continued:**
    + Storing data in state can lead to components being too encapsulated.
    This makes them difficult to work with.

---

#### Components > State

## General State tips

- If you need to use 'componentWillReceiveProps' to fit some data change into the component, refactor it to read data from the Redux store instead.

---

#### Components > State

- If the component uses state, but doesn't use any lifecycle methods, refactor it into a connected SFC.

---

#### Components > State

- If the component uses state AND lifecycle methods, refactor it to become a connected class component.

---

#### Components > State

## Summery:
+ Connected SFCs **>** `class` components utilizing state + lifecycle methods **>** `class` components only utilizing state.
+ State is good for local data such as the open state of a modal.

---

#### Components > State

+ It can be argued that even this data is better handled in your Redux code.
+ If a `class` component is using state, and you're forced to use `componentWillRecieveProps`, consider refactoring.

---

# In Conclusion
## Be excelent to each other and leave code a little better than you found it.

![](billted.jpeg)

