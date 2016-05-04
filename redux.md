# [Redux](http://redux.js.org/)

# [Redux Video Tutorials](https://egghead.io/lessons/javascript-redux-the-single-immutable-state-tree?series=getting-started-with-redux)

1. The entire state of the application is contained in a single Javascript object.
2. The state tree is read only. The only way to change the state tree is by dispatching an action, describing, in the minimum way, what changed in the application.
3. To describe state mutations, you have to write a function that takes the previous state of the app, the action being dispatched, and returns the next state of the app, and this function has to be pure. This function is called the *reducer*.

Pure functions: return value depends only on the value of arguments, with no side effects. Same inputs always produce same outputs. Arguments are never modified, instead return new object. This is a good reason to use an Immutable library.

## Reducers

* reducers must be pure functions; however, if part of the state doesn't change, the new state can contain a reference to the same object inside the old state.
* reducers must return the current state for any unknown action

### Reducer Composition

* Different reducers specify how different parts of the state tree are updated in response to actions. Reducers are also normal Javascript functions, so they can call other Reducers to delegate and abstract away handling of updates of some parts of the state they manage.
* Single top-level Reducer managing the state of the app, it is convenient to express it as many Reducers calling each other, each contributing to a part of the application state tree
* Redux has a `combineReducers` function for combining multiple Reducers into a single Reducer.
* It is good practice to name stateKeys the same as the Reducers that update their values. If you do this, you can even omit the Reducer names when using `combineReducers` thanks to ES6 object literal shorthand notation

## Testing

Testing using Michael Jackson's library called *expect*. Usage: `expect(...).toEqual(...)`

## Stores

The store binds together the 3 principles.
1. Holds the current application state object.
2. Lets you dispatch actions.
3. When you create it, you need to specify the reducer that tells how state is updated with actions.

Store has 3 methods:
1. `getState()` - retrieves the current state of the Redux store
2. `dispatch(action)` - dispatch actions that change the state of the application
3. `subscribe(function)` - register a callback that the Redux store will call any time an action has been dispatched

The Store can be implemented as an object containing the 3 functions above:
```
const createStore = (reducer) => {
    let state;
    let listeners = [];

    const getState = () => state;

    const dispatch = (action) => {
        state = reducer(state, action);
        listeners.forEach(listener => listener());
    };

    const subscribe = (listener) => {
        listeners.push(listener);
        // Function to unsubscribe other listeners
        return () => {
            listeners = listeners.filter(l => l !== listener);
        };
    };

    // Initialize store with default state
    dispatch({});

    return { getState, dispatch, subscribe };
};

// Examples
let store = createStore(myReducer);
store.subscribe(listener1); // 1
store.subscribe(listener2); // 1, 2
let unsubscribeOthers = subscribe(listener3); // 1, 2, 3
unsubscribeOthers(); // 3
subscribe(listener4)(); // 4
subscribe()(); // undefined
```

Uses a library called *deep freeze* to make sure that his code is free of mutations.

## Design

* *Presentation Components:* Treat React components as "presentation" components, by passing in onClick handlers as props instead of defining the logic inside the component itself
* *Container components:* defines the behaviors of the components and passes them in through props
    * ie. top level App components


## ReactRedux and Component Handling

* There is a library called *ReactRedux* that is made to simplify the use of context as well as Container Components
* takes care of reading the store from the context without needing to worry about contextTypes. This is great, because React recommends against using Context directly. It is "unstable" and "likely to change in the future".
* Import the `Provider` class to create the wrapper for the top-level app component that will contain the store in its context
* Use the `connect` method to create a Container component by defining and passing in `mapStateToProps` and `mapDispatchToProps` (these functions behave exactly like they sound), as well at the Presentation Component being wrapped. (It is a curried function so these are chained function calls).
  * you can think of this as "connecting" the Presentation component to the Redux store
  * the result of the `connect` call is a Container component that is going to render the Presentation component after calculating the props from the passed in functions
  * NOTE: you should keep every component in its own file to avoid needing to make the mapping functions with long unique names. With their own files, you can call them the same thing every time
* if there is no state to map to props, you can simply pass in `null` in place of the `mapStateToProps` function
* it is also very common to simply inject dispatch itself as a prop, so you can pass `null` in place of `mapDispatchToProps` to accomplish this
* if both functions are `null`, they can be removed and `connect()` can be called without any arguments at
  * result: do NOT subscribe to the store and inject the dispatch function as a prop
