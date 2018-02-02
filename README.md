# TypeSafeState-JS

[![npm version](https://badge.fury.io/js/type-safe-state-js.svg)](https://badge.fury.io/js/type-safe-state-js)
[![Build Status](https://travis-ci.org/protoman92/TypeSafeState-JS.svg?branch=master)](https://travis-ci.org/protoman92/TypeSafeState-JS)
[![Coverage Status](https://coveralls.io/repos/github/protoman92/TypeSafeState-JS/badge.svg?branch=master)](https://coveralls.io/github/protoman92/TypeSafeState-JS?branch=master)

### REACT NATIVE USERS, PLEASE READ THIS FIRST ###

Since React Native only allows the state to be a normal key-value object, using this State will lead to errors when we call **setState** on a Component:

> One of the sources for assign has an enumerable key on the prototype chain. Are you trying to assign a prototype property? We don't allow it, as this is an edge case that we do not support. This error is a performance optimization and not spec compliant.

The workaround for this issue is to convert the State to a normal KV object with **flatten** before setting state, and re-convert **this.state** back with **State.fromKeyValue** when we want to read data. When I define a Component, I usually do it as such:

```typescript
import { Component } from 'react';
import { State, StateType } from 'type-safe-state-js';

class App extends Component<Props.Type, StateType<any>> {
  public constructor(props: Props.Type) {
    super(props);
  }
  
  private operationThatAccessesState(): void {
    let a = State.fromKeyValue(this.state).valueAtNode('a.b.c.d');
    ...
  }
  
  private operationThatSetsState(state: State.Self<any>): void {
    this.setState(state.flatten());
  }
  
  private convertStateForPlatform(state: State.Self<any>): StateType<any> {
    return this.platform === REACT_NATIVE ? state.flatten() : state;
  }
}
```

Since **StateType** is defined as:

```typescript
type StateType<T> = State.Selt<T> | { [key: string] : T };
```

Using **StateType** as the state type for a component effectively takes care of both normal React.js and React Native. **State.fromKeyValue** checks whether the object is of class **State.Self** first before doing anything (and does nothing if it is), so we do not have to worry about unnecessary work.

### WHAT IS IT?

Functional, type-safe nested state object that can be used for (but not limited to) Redux architectures. Since it is immutable by default (all update operations must be done via a limited selection of functions), we do not need to worry about shared state management.

To use this State:

```typescript
import { State } from 'type-safe-state-js';
```

This State object contains the a key-value object of the current state values, as well as a key-value object of nested substates. To access the value at any node, use:

```typescript
state.valueAtNode(string);
```

The parameter of this function should be a String whose components are joined with the specified substateSeparator (which is by default '.'). For example:

```typescript
state.valueAtNode('a.b.c.d.e');
```

The above call will access the value at key **'e'** of substate **'a.b.c.d'**.

In order to update the value at some node, call:

```typescript
state.updatingValue(string, Nullable<any>);
```

The State object will update the value at that node, and if necessary create new substates along the way.

This State is useful for Redux reducers because you will not need to check for existence of property keys before updating value at a node. A reducer can be as such:

```typescript
function reduce(state: State.Self<any>, action: Action): State.Self<any> {
  return state
    .updatingValue('auth.login.username', action.username)
    .updatingValue('auth.login.password', action.password)
    .updatingValue('auth.login.email', action.email);
}
```

As a result, we have a robust, functional set of reducers.
