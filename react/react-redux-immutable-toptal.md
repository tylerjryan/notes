# [React, Redux, and Immutable.js: Ingredients for Efficient Web Applications](https://www.toptal.com/react/react-redux-and-immutablejs)

Project example code: https://github.com/rogic89/ToDo-react-redux-immutable

Most components can have `shouldComponentUpdate` equal to false, if possible. This ensures that the component will never re-render (except for the initial render), making the application extremely fast.

When we can't do this, the goal is to make an equality check of old props vs. new props as cheap as possible, and skip the re-render if the data is unchanged.

# How JavaScript performs equality checks for different data types

For primitive types like `boolean`, `string`, and `integer`, the check is very simple since they are always compared by their actual value:
```
1 === 1
'string' === 'string'
true === true
```

For complex types like `objects`, `arrays`, and `functions`, it is completely different:

Two `objects` are the same if they have the same reference (pointing to the same object in memory):
```
const obj1 = { prop: 'someValue' }
const obj2 = { prop: 'someValue' }
obj1 === obj2  // false
```

Therefore, if you compare complex types in the `shouldComponentUpdate` naively, the component will end up re-rendering itself anyway.

The important thing to note is that the data coming from Redux reducers, if not set up correctly, will always be served with difference references, which will cause components to re-render every time.

This is the core problem to avoiding unnecessary re-rendering!

# Handling References

For example: we have a deeply nested object, and we want to compare its current value to its previous. We could recursively loop through and compare each one, but that would be expensive, and is not an option.

The solution:
* preserving the reference if nothing has changed
* changing the reference if any of the nest object/array prop values changed

This is NOT easy, but Facebook realized this a long time ago and made **Immutable.js** to take care of it.

If you use Immutable data structures, you can do these direct comparisons and all the referencing is handled for you:
```
import { Map } from ‘immutable’;

//  transform object into immutable map
let obj1 = Map({ prop: ’someValue’ });  
const obj2 = obj1;
console.log(obj1 === obj2);  // true

obj1 = obj1.set(‘prop’, ’someValue’);  // set same old value
console.log(obj1 === obj2);  // true | does not break reference because nothing has changed

obj1 = obj1.set(‘prop’, ’someNewValue’);   // set new value
console.log(obj1 === obj2);  // false | breaks reference
```

None of the Immutable.js functions perform direct mutation on the given data. Instead, data is cloned internally, mutated and if there were any changes new reference is returned. Otherwise it returns the initial reference. New reference must be set explicitly, like obj1 = obj1.set(...);.

# Building an efficient application

* Build your state out of entirely Immutable objects for the complex types (ie. `List`, `Map`)
* No `.toJS()` required!
* use primitives directly

* [interesting] connect should be performed only on top level route components, extracting the data in `mapStateToProps` and the rest is basic React passing props to children.
  * on large scale applications, it tends to get hard to keep track of all the connections so we want to keep them to a minimum

Data to pass to Components:
* primitive types
* object/array only in immutable form

# Diff props in the simplest way possible

```
$ npm install react-pure-render
```

```
import shallowEqual from 'react-pure-render/shallowEqual';

shouldComponentUpdate(nextProps, nextState) {
  return !shallowEqual(this.props, nextProps) || !shallowEqual(this.state, nextState);
}
```

`shallowEqual` will check the props/state diff only 1 level deep. It works extremely fast.

But it would be very inconvenient to have to write this is every component. So just create special component containing this `shouldComponentUpdate` definition:
```
// components/PureComponent.js
import React from 'react';
import shallowEqual from 'react-pure-render/shallowEqual';

export default class PureComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    return !shallowEqual(this.props, nextProps) || !shallowEqual(this.state, nextState);
  }
}
```

Then just extend any component that you wish to contain this logic:
```
// components/Todo.js
export default class Todo extends PureComponent  {
  // Component code
}
```


## functions

when passing in functions as props, using the ES6 `class`, React does NOT automatically bind `this` to functions, so we have to do it manually. You can use the arrow function binding or use `bind` when calling the function. But both approaches will cause the component to re-render because the reference will change every time.

To fix this, you can pre-bind functions in the `constructor` method:
```
constructor() {
  super();
  this.handleClick = this.handleClick.bind(this);
}
// Then simply pass the function
render() {
  return <Component onClick={this.handleClick} />
}
```

# Handling immutable data inside a component

You now have immutable data inside your components, and you should NOT make this mistake of immediately converting it to a JavaScript object using `toJS()`. Instead, take advantage of the Immutable API!

## Chaining

If you need to chain together functions like
```
myMap.filter(...).sort(...)
```
...then it is VERY important to first convert it into `Seq` using `toSeq`, and then turn it back into desired form at the end:
```
myMap.toSeq().filter(somePred).sort(someComp).toOrderedMap()
```
`Seq` is lazy immutable sequence of data, meaning it will perform as few operations as possible to do its task while skipping creation of intermediate copies. `Seq` was built to be used this way.

If you want to shallowly convert `objects` and `arrays` (1 level deep) into plain JavaScript objects and arrays, the `toObject` and `toArray` methods work very well. Any data deeper than 1 level remaining in immutable form.  
