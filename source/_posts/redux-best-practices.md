---
title: Redux Best Practices
date: 2019-11-22 10:24:56
tags: redux, javascript, react, selectors, best practices
---

Redux is a predictable state management library that can be used with many different UI frameworks. When using with React, you use it along side of the tool `react-redux` in order to connect your components to the store and to be able to dispatch actions and retrieve state. After working on multiple projects in my career using Redux + the time I’ve spent personally using Redux, here are some tips on how to structure your redux store and code:

## Tips

1. Make your store as flat as possible. When you have deeply nested objects, you’ll find that your reducers will contain a lot of copying and checking and it’ll be an absolute pain to maintain this code as your store grows.

2. Leverage selectors as much as possible. Selectors allow you to compute state based on the application state. For example, imagine you have a list of accounts with a bunch of data associated with them. You can use a selector to build a new data structure for a component that may need that user list but in a different format. Selectors are memoized which means that all of your components which use a selector will only compute the value once and then it won’t update unless the store updates. The two main options are the `useSelector` hook that ships with `react-redux` and the library `reselect`.

3. Use actions creators — action creators are simply functions which return action objects. I’d recommend keeping with the standard of having just two properties at the top level of the action `type` and `payload` and then put inside the `payload` object whatever you want. Technically a Redux action is an object with at least one property `type` and you can put whatever else you want in it but I think it’s better to follow the standard flux action pattern. (Note: There's also a property you can add called `error` which will mark an action as an error action. Set this property to `true` if necessary)

4. Test your selectors and reducers — your selectors and reducers are where the vast bulk of your logic will exist and these should absolutely be tested.

5. I personally recommend keeping all of your reducers and actions in one file, alphabetized. I find trying to navigate the redux store and seeing what is available is difficult when you have so many different files being imported and magically combined in the end.

6. Your components should be as unaware of Redux as possible — from their perspective, they receive props and render them. They also receive actions via `mapDispatchToProps` and can execute them when needed. I’d avoid using `dispatch` as much as possible and instead create a shared function `mapDispatchToProps` which takes an `action` object (which contains all of your action creators) and then wraps it with `dispatch` — you can use the function `bindActionCreators` from `redux` to do this.

7. Leverage redux middleware like `redux-logic` or `rxjs` or similar to handle business logic or side effects from actions being dispatched. For example, I’ve used `redux-logic` before and are able to you can define a certain type of action that, when dispatched, will invoke a function. You can even configure this logic to execute before or after the action has reached the reducers. I put most of my asynchronous work here + redirect logic because my components should literally just be in charge of rendering the view and at the most, dispatching actions.

## Summary

Redux seems complicated but the reality is that the complexity comes from misunderstanding how it works and attempting to overengineer it from the beginning. It’s a store (an object), an action (an object), and a reducer (a pure function). People make stores and reducers absurdly complicated which make it difficult to follow what’s going on during the lifecycle of the application and splitting everything up across many different files makes it hard to understand what the store actually contains. Put business logic inside of middleware. Reducers just update state. Components render the view. If you follow the rules of the game, you’ll have a much easier time using redux and won’t find yourself completely lost on how anything is being done. Final note: install redux-devtools — it allows you to see all actions dispatched and how the state changes over time — it rules.
