# react-awesome 🐨

React can be really confusing if you are just starting out and learning from different sources all at the same time. So I decided to build my own opiniated guide to react ! This guide will contain a mix of good/standard practices and also my favorite ways of doing things and the reasons why.

## Separation between container and display containers 👣

Life is simpler when UI components are unaware of the network, business logic, or app state. Given the same props, always render the same data.

- **Container components**: components that are connected to the data store or local state and may have side-effects.
- **Presentation components**: mostly pure components, which, given the same props and context, always return the same JSX.

## Component state for component state 👓

Why ? 

- Don't use redux if you don't need it
- useState and useEffect hooks are often more then enough 

## Hooks

[hooks](https://reactjs.org/docs/hooks-intro.html)

Why ? 

- Use state and hook in the components lifecycle
- Replaces render prop pattern 
- Colocate related logic
- Share reusable behaviors independent of component implementations

## Testing  🦢

preferred lib: [react-testing](https://testing-library.com/docs/react-testing-library/intro)

Why ? 

- Test behavior of components instead of implementation
- Easily test with nodes using `data-testid`
- Encourages best practices



## State container for Application wide state 🕶

preferred lib: [react-redux](https://react-redux.js.org)

Why ? 

- Deterministic state resolution (enabling deterministic view renders when combined with pure components)
- Transactional state
- Isolate state management from I/O and side-effects
- Single source of truth for application state
- Easily share state between different components
- Transaction telemetry (auto-logging action objects)
- Time travel debugging

[react-redux](https://react-redux.js.org/api/hooks) with hooks

Why ? 

- Subscribe to the Redux store and dispatch actions, without having to wrap your components in connect()
- Useful for using dispatch in deeply nested components 

```javascript
import React, { useCallback } from 'react'
import { useDispatch } from 'react-redux'

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch()
  const incrementCounter = useCallback(
    () => dispatch({ type: 'increment-counter' }),
    [dispatch]
  )

  return (
    <div>
      <span>{value}</span>
      <MyIncrementButton onIncrement={incrementCounter} />
    </div>
  )
}

export const MyIncrementButton = React.memo(({ onIncrement }) => (
  <button onClick={onIncrement}>Increment counter</button>
))
```

## Computing Derived Data

[reselect](https://github.com/reduxjs/reselect)

Why ? 

- Selectors can compute derived data, allowing Redux to store the minimal possible state.
- Selectors are efficient. A selector is not recomputed unless one of its arguments changes.
- Selectors are composable. They can be used as input to other selectors.

```javascript
import { createSelector } from 'reselect'

const shopItemsSelector = state => state.shop.items
const taxPercentSelector = state => state.shop.taxPercent

const subtotalSelector = createSelector(
  shopItemsSelector,
  items => items.reduce((acc, item) => acc + item.value, 0)
)

const taxSelector = createSelector(
  subtotalSelector,
  taxPercentSelector,
  (subtotal, taxPercent) => subtotal * (taxPercent / 100)
)

export const totalSelector = createSelector(
  subtotalSelector,
  taxSelector,
  (subtotal, tax) => ({ total: subtotal + tax })
)

let exampleState = {
  shop: {
    taxPercent: 8,
    items: [
      { name: 'apple', value: 1.20 },
      { name: 'orange', value: 0.95 },
    ]
  }
}

console.log(subtotalSelector(exampleState)) // 2.15
console.log(taxSelector(exampleState))      // 0.172
console.log(totalSelector(exampleState))    // { total: 2.322 }
```


## Composed Selectors

### composedSelectors.js

```javascript
import { createSelector } from 'reselect'

const getBar = (state) => state.foo.bar

export const getBarState = createSelector(
    [getBar],
    (bar) => bar
)

export const firstSelector = (state) => state.a
export const secondSelector = (state) => state.b
export const thirdSelector = (state) => state.c

export const myComposedSelector = createSelector(
  firstSelector,
  secondSelector,
  thirdSelector,
  (a, b, c) => a * b * c
)
```

### __tests__/composedSelectors.js

```javascript
import { getBarState, firstSelector, secondSelector, thirdSelector, myComposedSelector } from "../composedSelectors";

describe('selectors', () => {
    it('should select bars state', () => {
        expect(getBarState({ foo: { bar: [0] } })).toEqual([0])
    })

    it('should select a state', () => {
        expect(firstSelector({ a: 1 })).toEqual(1)
    })

    it('should select b state', () => {
        expect(secondSelector({ b: 1 })).toEqual(1)
    })

    it('should select c state', () => {
        expect(thirdSelector({ c: 1 })).toEqual(1)
    })

    it('should calculate composed selector', () => {
        expect(myComposedSelector.resultFunc(2,2,2)).toEqual(8)
    })
})
```

## Redux state normalization

Nested data means that the corresponding reducer logic has to be more nested or more complex. In particular, trying to update a deeply nested field can become very ugly very fast.

Since immutable data updates require all ancestors in the state tree to be copied and updated as well, and new object references will cause connected UI components to re-render, an update to a deeply nested data object could force totally unrelated UI components to re-render even if the data they're displaying hasn't actually changed.

[Normalizing state shape redux documentation](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape)

```javascript
import { combineReducers } from "redux"


const rootReducer = combineReducers({
  users: sectionReducer("USER")(users),
  articles: sectionReducer("ARTICLE")(articles)
})
```



## Component local side effects 🏄🏽

[useEffect hook](https://reactjs.org/docs/hooks-effect.html)

Why ? 

- Remove duplicates between lifecycle methods (componentDidMount, componentDidUpdate, etc.)

```javascript
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

## Application wide side effects 🏋

preferred lib: [redux-saga](https://redux-saga.js.org/)

A redux middleware that uses generators to apply side effects after a redux action has been dispatched to the store. 

Why ? 

- Complex async side effects (fork, race, channels, etc.)
- Compose parallel tasks
- Easily test generators
- **Have full power over how & when effects are executed**

## Forms 📝 

preferred lib: [react-hook-form](https://react-hook-form.com/)

Why ? 

- Compact Code
- Isolates Component Re-renders
- Input change subscriptions
- Faster mounting

```javascript
import React from "react";
import { useForm } from "react-hook-form";

const Example = () => {
  const { handleSubmit, register, errors } = useForm();
  const onSubmit = values => console.log(values);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        name="email"
        ref={register({
          required: "Required",
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: "invalid email address"
          }
        })}
      />
      {errors.email && errors.email.message}

      <input
        name="username"
        ref={register({
          validate: value => value !== "admin" || "Nice try!"
        })}
      />
      {errors.username && errors.username.message}

      <button type="submit">Submit</button>
    </form>
  );
};
```

## Testing Forms

### Form to test

```javascript
import React from "react";
import { useForm } from "./src";

import "./styles.css";

export default function App() {
  const { register, handleSubmit, watch, errors } = useForm();
  const onSubmit = data => {
    console.log(data);
  };

  const example = watch("example");

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <label>Example</label>
      <input
        name="example"
        defaultValue="test"
        ref={register}
        data-testid="example"
      />
      <label>ExampleRequired</label>
      <input
        name="exampleRequired"
        ref={register({ required: true, maxLength: 10 })}
        data-testid="exampleRequired"
      />
      {errors.exampleRequired && <p>This field is required</p>}

      {example === "test" && <i data-testid="message">Hidden message</i>}
      <input type="submit" data-testid="submit" />
    </form>
  );
}
```

### Tests

```javascript
import React from "react";
import App from "./App";
import { render, fireEvent } from "@testing-library/react";

describe.only("App", () => {
  test("should watch input correctly", () => {
    const { getByTestId } = render(<App />);

    fireEvent.input(getByTestId("example"), {
      target: {
        value: "test"
      }
    });

    expect(getByTestId("message").innerHTML).toEqual("Hidden message");
  });

  test("should display correct error message", () => {
    const { getByTestId, findByText } = render(<App />);

    getByTestId("submit");

    fireEvent.click(getByTestId("submit"));

    findByText("This field is required");
  });
});
```

## Routing ⚒

preferred lib: [react-router](https://reactrouter.com/web/guides/quick-start)

Why ? 

- Dynamic Routing 
- Nested Routes
- Responsive Routes

```javascript
import React, { Component } from 'react';
import { BrowserRouter as Router, Route } from 'react-router-dom'
import Footer from './components/Footer';

class App extends Component {
  render() {
    return (
      <div>
        <Router>
          <div>
            <h1>React router</h1>
            <Route path='/:filter?' render={({match}) => (
              <h2>{match.params.filter}</h2>
            )} />
            <Footer/>
          </div>
        </Router>
      </div>
    )
  }
}
```

## Optimistic Updates

```javascript
import React, {Component} from 'react';

const Items = ({ items, deleteItemOptimistic, loading }) => {
    return (
        <ul style={{ opacity: loading ? 0.6 : 1 }}>
            {items.map(item => (
                <li key={item.id}>
                    {item.title}{' '}
                    <button onClick={() => deleteItemOptimistic(item.id)}>
                        Delete item
              </button>
                </li>
            ))}
        </ul>
    )
}

function deleteItemRequest(id) {
  return new Promise((resolve, reject) => {
    setTimeout(id === 3 ? reject : resolve, 750);
  });
}

class App extends Component {
  state = {
    items: Array.from(Array(5), (_, i) => ({
      id: i + 1,
      title: `Item ${i + 1}`,
    })),
    loading: false,
    error: null,
  };

  deleteItemOptimistic = id => {
    const deletingItem = this.state.items.find(item => item.id === id);

    this.setState(state => ({
      items: state.items.filter(item => item.id !== id),
    }));

    deleteItemRequest(id)
      .catch(() =>
        this.restoreItem(deletingItem, id)
      );
  };

  restoreItem(deletingItem, id) {
    return this.setState(state => ({
      items: [...state.items, deletingItem].sort((a, b) => a.id - b.id),
      error: `Request failed for item ${id}`,
    }));
  }

  render() {
    const {items, loading, error} = this.state;

    return (
      <div>
        <h4>Optimistic UI updates in React using setState()</h4>
        <Items loading={loading} deleteItemOptimistic={this.deleteItemOptimistic} items={items} />
        {error && <p>{error}</p>}
      </div>
    );
  }
}
```
