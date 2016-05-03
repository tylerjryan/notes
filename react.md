# React 

# [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html)

Steps:
1. Break the UI into a component hierarchy
2. Build a static version of the UI with no interactivity
  * this includes creating your classes and writing render functions
  * data is passed using `props` from parents to children
  * don't use state at all! State is reserved for interactivity
  * at the end of this step, you have a library of reusable components that render your data model
  * the component at the top of the hierarchy will take your data model as a prop
3. Identify the minimal (but complete) representation of UI state  
  * ask 3 questions about each piece of data you think you have:
    * is it passed in from a parent via props? If so, it probably isn't state.
    * does it change over time? If not, if probably isn't state.
    * can you compute it based on any other state of props in your component? If so, it's not state.
4. Identify where your state should live
  * remember that React is about one-way data flow
  * For each piece of state in your application:
    * Identify every component that renders something based on that state.
    * Find a common owner component (a single component above all the components that need the state in the hierarchy).
    * Either the common owner or another component higher up in the hierarchy should own the state.
    * If you can't find a component where it makes sense to own the state, create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component.
5. Add inverse data flow
  * components deep in the hierarchy will need to update the state of things above
  * to do this, the parents can pass callbacks to children to be called when state changes

# [Displaying Data](https://facebook.github.io/react/docs/displaying-data.html)

JSX is a Javascript syntax extension taht looks similar to XML. JSX lets you create Javascript objects using HTML syntax. It is completely optional however.

You can also create elements using pure Javascript like this:
```
React.createElement('a', {href: 'https://facebook.github.io/react/'}, 'Hello!')
```

# [JSX In Depth](https://facebook.github.io/react/docs/jsx-in-depth.html)

## HTML Tags vs. React Components

React can either render HTML tags (strings) or React components (classes).

React uses upper vs. lowercase tags names to distinguish between custom React components and HTML tags.

For React components, use a local variable that stars with an uppercase letter:
```
var MyComponent = React.createClass({/*...*/});
var myElement = <MyComponent someProperty={true} />;
ReactDOM.render(myElement, document.getElementById('example'));
```
For HTML tags, you must use LOWERCASE tag names in JSX.
```
var myDivElement = <div className="foo" />;
ReactDOM.render(myDivElement, document.getElementById('example'));
```

## Transformation

React JSX transforms an XML-like syntax into native Javascript. Here are some examples:
```
var Nav;
// Input (JSX):
var app = <Nav color="blue" />;
// Output (JS):
var app = React.createElement(Nav, {color:"blue"});

// You can also specify children:
var Nav, Profile;
// Input (JSX):
var app = <Nav color="blue"><Profile>click</Profile></Nav>;
// Output (JS):
var app = React.createElement(
  Nav,
  {color:"blue"},
  React.createElement(Profile, null, "click")
);

// JSX will infer the class's displayName from the variable assignment when the displayName is undefined.
// Input (JSX):
var Nav = React.createClass({ });
// Output (JS):
var Nav = React.createClass({displayName: "Nav", });
```

## Namespaced components

If you are building something with many children (ie. a form), you can use *namespaced components* that can contain other components as attributes.

Example:
```
var Form = MyFormComponent;

var App = (
  <Form>
    <Form.Row>
      <Form.Label />
      <Form.Input />
    </Form.Row>
  </Form>
);

var MyFormComponent = React.createClass({ ... });

MyFormComponent.Row = React.createClass({ ... });
MyFormComponent.Label = React.createClass({ ... });
MyFormComponent.Input = React.createClass({ ... });
```

## Javascript Expressions

Javascript expressions can be used as attribute values in React components. To use a Javascript expression, wrap it in a pair of curly braces `{}` instead of quotes.

Example:
```
// Input (JSX):
var person = <Person name={window.isLoggedIn ? window.name : ''} />;
// Output (JS):
var person = React.createElement(
  Person,
  {name: window.isLoggedIn ? window.name : ''}
);
```

## Boolean attributes

Omitting the value of an attribute causes JSX to treat it as true. To pass false an attribute expression must be used. This often comes up when using HTML form elements, with attributes like disabled, required, checked and readOnly.

```
// These two are equivalent in JSX for disabling a button
<input type="button" disabled />;
<input type="button" disabled={true} />;

// And these two are equivalent in JSX for not disabling a button
<input type="button" />;
<input type="button" disabled={false} />;
```

## Child Expressions

Javascript expressions can also be used to express children.

Example:
```
// Input (JSX):
var content = <Container>{window.isLoggedIn ? <Nav /> : <Login />}</Container>;
// Output (JS):
var content = React.createElement(
  Container,
  null,
  window.isLoggedIn ? React.createElement(Nav) : React.createElement(Login)
);
```

# [JSX Spread Attributes](https://facebook.github.io/react/docs/jsx-spread.html)

If you know all the properties that a component will have ahead of time, you can use JSX and define them:
```
var component = <Component foo={x} bar={y} />;
```

**Mutating Props is Bad**: If you don't know the properties ahead of time, you might be tempted to define your component and then add the props later, like this:
```
// BAD, don't do this
var component = <Component />;
component.props.foo = x; // bad
component.props.bar = y; // also bad
```

If you do this, React can't check for the right propTypes until way later, so your errors end up with a cryptic stack trace.

*Props should be considered immutable*.


## Spread Attributes

Spread attributes allow you to expand an objects key/value pairs into the attributes of a component. For example:
```
var props = {};
props.foo = x;
props.bar = y;
var component = <Component {...props} />;
```

Note that you can combine a spread attribute unpacking with normal attribute passing. But props that come later will override props that were previously defined. For example:
```
var props = { foo: 'default' };
  var component = <Component {...props} foo={'override'} />;
  console.log(component.props.foo); // 'override'
```


# [JSX Gotchas](https://facebook.github.io/react/docs/jsx-gotchas.html)

JSX looks like HTML but there are some important differences:

## HTML Entities

You can insert HTML entities into literal text. But if you want to include HTML entites inside of Javascript expressions `{}`, you will run into "double escaping" issues because React escapes all strings in order to prevent XSS attacks.

For example, this works:
```
<div>First &middot; Second</div>

```
...but this doesn't:
```
// Bad: It displays "First &middot; Second"
<div>{'First &middot; Second'}</div>
```

To work around this, you should find the unicode number of the entity and use it inside a Javascript string:
```
<div>{'First \u00b7 Second'}</div>
<div>{'First ' + String.fromCharCode(183) + ' Second'}</div>
```

You can also use mixed arrays of string and JSX elements:
```
<div>{['First ', <span>&middot;</span>, ' Second']}</div>
```

As a last resort, you can insert raw HTML:
```
<div dangerouslySetInnerHTML={{__html: 'First &middot; Second'}} />
```

## Custom HTML Attributes

You can pass properties to native HTML elements that aren't in the HTML specification, but they won't be rendered.

# [Interactivity and Dynamic UIs](https://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html)

With React you pass your event handler as a camelCased prop similar to how you'd do it in normal HTML. React ensures that all events behave similarly in all browsers by implementing a synthetic event system. It ensures consistency with the W3C spec.

## Autobinding and Event Delegation

React automatically binds each method to its component instance, and caches the method such that it's extremely CPU and memory efficient.

When React starts up, it listens for all events at the top level using a single event listener. When a component is mounted or unmounted, the event handlers are simply added or removed from an internal mapping. When an event occurs, React knows how to dispatch it using this mapping.

## Components are just state machines

React thinks of UIs as simple state machines. By thinking of a UI as being in various states and rendering those states, it's easy to keep your UI consistent.

In React, you simply update a component's state, and then render a new UI based on this new state. React takes care of updating the DOM for you in the most efficient way.

A common way to inform React of a data change is by calling `setState(data, callback)`. This method merges `data` into `this.state` and re-renders the component. When the component finishes re-rendering, the optional `callback` is called. Most of the time you'll never need to provide a `callback` since React will take care of keeping your UI up-to-date for you.

## What components should have state?

Most components should simply take some data from props and render it. But sometimes you need to respond to user input, a server request, or the passage of time. State is used for these.

**Try to keep as many of your components as possible stateless**.

A common pattern is to create several stateless components that just render data, and have a stateful component above them in the hierarchy that passes its state to its children via props. The stateful component encapsulates all of the interaction logic, while the stateless components take care of rendering data in a declarative way.


## What *should* go in state?

**State should contain data that a component's event handlers may change to trigger a UI update.** Inside of a component's `render` function, you should compute any other information you need based on `this.state`.

## What *shouldn't* go in state?

`this.state` should only contain the **minimal** amount of data needed to represent your UI's state. Therefore, it should NOT contain:

* **computed data:** don't worry about precomputing values based on state. It is easy to ensure the UI is consistent if you do all your computation in `render()`.
* **React components:** Build then in `render()` based on underlying props and state.
* **Duplicated data from props:** Try to use props as the *source of truth* where possible. One valid use to store props in state is to be able to know its previous values, because props may change as the result of a parent component re-rendering.


# [Multiple Components](https://facebook.github.io/react/docs/multiple-components.html)

Rendering single components is nice, but a big feature of React is **composability**.

## Motivation: Separation of Concerns

You can separate concerns of your app by building components, and have components make use of one another through well-defined interfaces.

## Ownership

A React component can use other components in its `render` function. In React, **an owner is the component that sets the `props` of other components**. Props are always consistent with what the owner sets, and cannot be mutated.

There is a difference between the owner-ownee and parent-child relationship. Parent-child is the standard relationship in the DOM, where an HTML element has children elements inside it. This is does not have any implications for ownership because there is no relationship to prop setting.

## Children

When creating a React component instance, you can put React components inside one another as children. For example:
```
<Parent><Child /></Parent>
```
`Parent` does NOT own `Child`. If the above was in the `render()` of a class called `Master`, then `Master` would be the owner of both `Parent` and `Child`.

But `Parent` can access its children through a special prop called `this.props.children`. You can work with children through utilities [here](https://facebook.github.io/react/docs/top-level-api.html#react.children).

## Child Reconciliation

**Reconciliation is the process by which React updates the DOM with each new render pass**. React will reconcile changes in the DOM according to the order of the children. See example:
```
// Render Pass 1
<Card>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
// Render Pass 2
<Card>
  <p>Paragraph 2</p>
</Card>
```

It may be intuitive that `<p>Paragraph 1</p>` was removed. But actually the text of the first `<p>` is changed to "Paragraph 2" and the second `<p>` is destroyed.

## Stateful Children

The above is not a big deal for most components, but for stateful components that maintain data in `this.state` across render passes, this can be very problematic. In most cases, this can be avoided by hiding elements instead of destroying them:
```
// Render Pass 1
<Card>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
// Render Pass 2
<Card>
  <p style={{display: 'none'}}>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
```

## Dynamic Children

CONTINUE HERE.


## Props

Immutable properties of a React component. They are set in the properties of the HTML component
in the render function of a parent component.

For example, the props of a "Note" component would be set in the render function of the "Notes"
component like this:
```
<Note task={note.task}
    onEdit={onEdit.bind(null, note.id)}
    onDelete={onDelete.bind(null, note.id)}/>
```

Then in the constructor of "Note", `task`, `onEdit`, and `onDelete` are set.

## State

State contains mutable characteristics that affect the rendering of a component. The state can
be initialized using `this.state = {...}` in the constructor of a component; the state can be
accessed using `this.state.variable`, and the state can be updated using `this.setState({key: value, ...})`.

**NOTE**: in Stores, the instance variables become the state. This is different from a normal
React Component where the state is explicitly initialized using this.state = {...}

## Linting

* Use ESLint with React plugin: `npm i eslint eslint-plugin-react --save-dev`
* add to `package.json`:
```
"scripts": {
  ...
  "lint": "eslint . --ext .js --ext .jsx"
}
...
```
* Generate a sample .eslintrc using eslint --init


## Component lifecycle

* componentWillMount() gets triggered once before any rendering. One way to use it would be to load data asynchronously there and force rendering through setState.
* componentDidMount() gets triggered after initial rendering. You have access to the DOM here. You could use this hook to wrap a jQuery plugin within a component, for instance.
* componentWillReceiveProps(object nextProps) triggers when the component receives new props. You could, for instance, modify your component state based on the received props.
* shouldComponentUpdate(object nextProps, object nextState) allows you to optimize the rendering. If you check the props and state and see that there's no need to update, return false.
* componentWillUpdate(object nextProps, object nextState) gets triggered after shouldComponentUpdate and before render(). It is not possible to use setState here, but you can set class properties, for instance. The official documentation goes into greater details. In short, this is where immutable data structures, such as Immutable.js, come handy thanks to their easy equality checks.
* componentDidUpdate() is triggered after rendering. You can modify the DOM here. This can be useful for adapting other code to work with React.
* componentWillUnmount() is triggered just before a component is unmounted from the DOM. This is the ideal place to perform cleanup (e.g., remove running timers, custom DOM elements, and so on).


## Create class

Beyond the lifecycle hooks, there are a variety of properties and methods you should be aware of if you are going to use React.createClass:

displayName - It is preferable to set displayName as that will improve debug information. For ES6 classes this is derived automatically based on the class name.
getInitialState() - In class based approach the same can be achieved through constructor.
getDefaultProps() - In classes you can set these in constructor.
mixins - mixins contains an array of mixins to apply to components.
