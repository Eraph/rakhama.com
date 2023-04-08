---
title: "Fluxor for Dummies"
description: "A very brief guide to Flux principles in the context of Fluxor."
date: 2023-04-06T15:44:33+10:00
draft: false
categories:
  - Programming
tags:
  - blazor
  - future-reference
  - programming
  - .net
  - tips
series:
  - Professional Development
---
[Fluxor](https://github.com/mrpmorris/Fluxor) is a Flux implementation for Blazor in the vein of Redux. It is a state management system designed for larger software systems that encourages immutability of state, pure functions and clean separation of code. Given my fairly basic understanding of the Flux pattern I thought it would be valuable to document what it is at a high level and what's required when using Fluxor to implement the various concepts.

<!--more-->
Note that this is not a tutorial on Flux or Fluxor, both of those are covered quite nicely in the [Fluxor Documentation](https://github.com/mrpmorris/Fluxor/tree/master/Docs#tutorials) already. This is my attempt to distill the two into a form that I at least will find useful to refer to. For more detailed implementation information, refer to the aforementioned tutorials.

## Flux Concepts
- **Dispatcher** - responsible for directing actions and updating state.
- **State** - a subsection of the store, per domain. Immutable.
  - State classes/records in Fluxor have the `[FeatureState]` attribute.
- **Action** - Classes that are created when an action is to be performed, consumed by Effects and Reducers. May be empty but typically contain the results of an action.
- **Reducer** - triggered by an action, creates a new state usually based on the previous state and sometimes with the results of the action.
  - Reducer methods are typically `static` to encourage pure functions, and have the `[ReducerMethod]` attribute in Fluxor.
- **Effect** - triggered by an action, typically performs an activity such as a service request and then could trigger another action. Does not affect state.
  - Effect methods can be static but will need to be part of a class instance if they depend on services which will be injected. Effect methods have the `[EffectMethod]` attribute in Fluxor.
- **ActionSubscriber** - a service (`IActionSubscriber`) that intercepts actions independent of the store.
- Blazor components and pages should inherit `Fluxor.Blazor.Web.Components.FluxorComponent`.
  - If unable to inherit from that class, subcribe to the `StateChanged` event and execute `InvokeAsync(StateHasChanged)`. Don't forget to unsubscribe!

## In Practice
As an example, the CLI tutorial given on Fluxor's docs does the following:
  - Prompts the user to select an action
  - User requests a weather update
    - An **action** is created to fetch the weather
      - The **action** is intercepted by a **reducer** which updates the **state**, emptying the weather list and sets Loading to true
      - The **action** is also intercepted by an **effect** which makes an API call to get the weather
        - When complete, the **effect** raises another **action** with the results
          - The **action** is intercepted by another **reducer** which updates the **state** with the results and sets Loading to false

I've taken note of this as I particularly like the way the initial action is used to update the state by way of a reducer while also triggering the effect to load the data.