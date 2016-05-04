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

## Container components

* Container components: job is to connect a presentation component to the Redux store, and specify the data and behavior that it needs
* The goal is to be able to use the component anywhere without needing to pass in any additional data to the component that contains it. You can verify the implementation by confirming that no props are provided to the container component where it is used.
* They are implemented as classes that take in props and access the state directly from `store.getState()`. Additionally, they must subscribe to the store directly, otherwise the component will not necessarily re-render when the app state changes. Ex:
```
class VisibleTodoList extends Component {
    componentDidMount() {
        // Use the return value to get the unsubscribe method
        // use the React built-in forceUpdate to force an update any time the store state changes
        this.unsubscribe = store.subscribe(() => this.forceUpdate());
    }

    componentWillUnmount() {
        this.unsubscribe();
    }

    render() {
        const props = this.props;
        const state = store.getState();

        return (
            <TodoList ...
                ...
            />
        );
    }
}
```
* You shouldn't treat the presentation/container architecture as a requirement. Only do this when it truly reduces the complexity of the code base. In general, start by trying to just extract the presentational components. If there is too much boilerplate from passing props down the tree, then create containers around them to load the data and specify the behavior

## Passing the store explicitly through props

* a global `store` variable works fine in a single-file example, but doesn't scale out to real apps for a few reasons
  * makes container components harder to test because they reference a specific store, and you may want to use a mock store in a test
  * makes it very hard to implement universal applications that are rendered on the server, because on the server, you want to supply a different store instance for every request, because each request has different data.
* in `componentDidMount`, get the store: `const { store } = this.props;`
* if you simply pass the store creation as a prop to the top level component, you'll end up passing it as a prop all the way down the tree, and this is very inconvenient. But there is a better way: passing the store implicitly

## Passing the store implicitly via Context

* there is another way to pass the store using the advanced React feature called *Context*
* create a `Provider` class that renders its children. This class with implement Context to make the store available to all of its children, including grandchildren
* the top level app component will then be included as a child of the `Provider`, and therefore every component in the app will have access to the store. It does this using a method called `getChildContext()`, which will be called by React.
* For this to work, you must specify `childContextTypes` for the component that defines `getChildContext` (see example). These are essential for the Context to be turned on. These are similar to normal propTypes but are required. For more on propTypes, see [here](https://facebook.github.io/react/docs/reusable-components.html).
```
class Provider extends Component {
  getChildContext() {
    return {
      store: this.props.store
    };
  }

  render() {
    return this.props.children
  }
}
Provider.childContextTypes = {
  store: React.PropTypes.object
};
...
const { createStore } = Redux;

ReactDOM.render(
  <Provider store={createStore(appReducer)}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```
* for a component to receive the relevant context, it must specify `contextTypes` like this:
```
ComponentClassName.contextTypes = {
  store: React.PropTypes.object
};
```
* any component that meets the above will therefore receive the Context and can extract it like this:
  `const { store } = this.context` or `const store = this.context.store`
* for functional components, the Context will be passed as a second argument after the props, and you can extract the store directly:
```
const AddTodo = (props, { store }) => { // args same as (props, context)
  ...
  return (
    ...
    )
}
```

## React-Redux and Component Handling

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
