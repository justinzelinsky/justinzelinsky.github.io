---
title: How I've changed my coding habits
date: 2020-05-13 21:34:26
tags:
  - best practices
  - habits
  - preferences
  - javascript
---

I've been a professional software engineer for almost seven years. When I reflect on how much I've changed and how much I've learned over the years, I'm astonished. Sometimes I will review code I've written not one year ago and be ashamed of how poor my code was -- it was too verbose or too "hacky" or too "clever." At my current job, I've found myself in a new environment -- I currently work on an enterprise application written in Ember over the course of 5 years by multiple teams located across the entire world. I've gained a new appreciation for opinionated frameworks and I've learned how to become a useful engineer and help deliver code which may not be the sexiest but satisfies our customers.

Over this time, my preferences for writing JavaScript have changed. I've realized certain traps I've fallen into and I've started to reassess some of the patterns that I felt were right to follow when coding.

In this post, I hope to outline some of the coding changes I've made and I hope you see the value in why I made these changes. I intend to update it with new items as they come to mind.

## `() => {}` versus `function`

When I learned about [arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) in my career, I fell in love. I appreciated the brevity:

```javascript
const relevantItems = myArr.filter(function(item) {
  return item.foo === 'bar';
});
```

versus

```javascript
const relevantItems = myArr.filter(item => item.foo === 'bar');
```

It seemed so obvious to me that there were hardly any situations where arrow functions wouldn't be preferably to use. I had recrntly been for all intents and purposes a React engineer -- not a front-end engineer. I worked in React. I had React solutions to any problems. My current job forced me to reassess everything when I had to forgo the freedom of React and embrace the order of Ember.

During my React times, I was writing components using the arrow function syntax. 

e.g.

```javascript
const MyComponent = props => {
  const { someProp, triggerEvent } = props;

  const handleClick = () => {
    triggerEvent(someProp);
  };

  return (
    <div className="my-component">
      <button onClick={handleClick}>Click me!</button>
    </div>
  )
}
```

My components were arrow functions, my functions were arrow functions. Even before I left behind class components and embraced function components à la hooks, I had still used arrow functions on my class components.

e.g.

```javascript

class MyOtherComponent extends React.Component {
  state = {
    foo: ''
  },

  changeFoo = () => {
    this.setState({ foo: Math.random() });
  }

  render() {
    const { foo } = this.state;

    return (
      <div className="my-other-component">
        <button onClick={this.changeFoo}>Change something!</button>

        Foo is: {foo};
      </div>
    )
  }
}
```

In fact, to be completely honest, I didn't realize I was actually using class properties for the longest time to allow for arrow functions on class definitions. I was all about arrow functions. Forget about `this.foo = this.foo.bind(this);`!

However, as I've found myself in the Ember world, I started writing components differently. We were on an older version of Ember (something which seems to be commonplace at large institutuons). My components looked like this:

```javascript

const MyComponent = Component.extend({
  bar: service(),

  foo() {
    const bar = this.get('bar');

    const baz = bar.doSomething();

    return baz;
  }
});
```

I was stuck in the world of `this` once again. However, being back in the world of `this`, I started to reassess my use of arrow functions versus standard function declarations. There is certainly a time and place where `this` matters and using arrow functions is certainly preferable to the classic `const self = this;`. That being said, I started to realize that my desire to always use arrow functions actually signaled some intent which may or may not be true.

Arrow functions do not have "its own bindings to the this, arguments, super, or new.target keywords." When I use arrow functions, I am actually signaling that the various bindings may actually matter which may or may not be true. The actual function keyword works perfectly fine I realized. Even when I'm not writing object-oriented code, it may be useful to use the actual function declaration and leverage arrow functions when I need the different behavior of arrow functions (e.g. infering the value of `this` or being able to have implicit returns for one-liners.)

### tl;dr

I now prefer to use `function` over `() => {}` unless I actually need the benefits of arrow functions.

## Opinions are not that bad

***WIP***


