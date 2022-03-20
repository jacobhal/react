# React

This is a repository for the Udemy course "React - The Complete Guide (incl Hooks, React Router, Redux)". This repository README contains general tips and notes that I like to keep in mind. If you navigate to the folders you will see the READMEs of each section of the course. The outer level also contains the source code for the projects that were created in this course.

- [React - The Complete Guide (incl Hooks, React Router, Redux)](#react---the-complete-guide-incl-hooks-react-router-redux)
  - [Certificate](#certificate)
  - [Forms in React](#forms-in-react)
  - [Async](#async)
    - [Await/async in React](#awaitasync-in-react)
      - [Parallel fetch requests](#parallel-fetch-requests)
    - [setState Asynchronous](#setstate-asynchronous)
  - [Lazy Initial State](#lazy-initial-state)
  - [React.Fragment](#reactfragment)
  - [Decoupling](#decoupling)
    - [Custom Hooks](#custom-hooks)
    - [Render props](#render-props)
    - [Higher Order Components (HOCs)](#higher-order-components-hocs)
  - [The "key" value](#the-key-value)
  - [Controlled Components](#controlled-components)
  - [Dynamic imports](#dynamic-imports)
  - [React.lazy](#reactlazy)
  - [React.memo](#reactmemo)
  - [useCallback](#usecallback)
  - [useRef](#useref)
  - [useContext](#usecontext)
  - [useReducer](#usereducer)
  - [useMemo](#usememo)
  - [Repeatable components](#repeatable-components)

## Certificate

TODO

## Forms in React

Form handling in React is a well discussed topic. There are many different popular libraries for form handling and validation such as React Hook Form. In order to split larger forms into multiple components, you can pass an onChange handler to the forms' children components and update the full form state in that way. Below is a rather extensive example of such an implementation with React and Typescript:

```TSX
// AppInterface.tsx
export interface IDefaultData {
  schools: string[];
}

export interface IFormState {
  personalData: IPersonalData;
}

export interface IPersonalData {
  name: string;
  gender: string;
  hasJob: boolean;
  hasChidren: boolean;
  school: string;
}
```

```TSX
// App.tsx
import { Form, Button } from 'semantic-ui-react';
import React, { useState, useEffect } from 'react';
import { AppContext } from './AppContext';
import { IPersonalData, IDefaultData, IFormState } from './AppInterface';
import PersonalData from './PersonalData';

const App: React.FC = () => {
    // Fetch baseUrl from a global AppContext
    const { baseUrl } = React.useContext(AppContext);

    const [defaultData, setDefaultData] = useState<IDefaultData>({
        schools: [],
        name: ''
    });

    const [formState, setFormState] = useState<IFormState>();

    useEffect(
        () => {
            let unmounted = false;

            // Fetch data that we need to prefill some components on page load
            const fetchData = async () => {
                try {
                    const response = await fetch(`${baseUrl}/api/default-data`);

                    if (response.ok) {
                        const data = await response.json();
                        !unmounted && setDefaultData(data);
                    }
                } catch (error) {
                    // eslint-disable-next-line no-console
                    console.log('Could not fetch data');
                }
            };

            fetchData();

            // return a cleanup function to avoid state updates after unmount
            return () => {
                unmounted = true;
            };
        },
        // eslint-disable-next-line react-hooks/exhaustive-deps
        []
    );

    const printForm = async (event: React.FormEvent) => {
        event.preventDefault();
        const requestOptions = {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(formState),
        };

        try {
            const response = await fetch(`${baseUrl}/api/print`, requestOptions);

            if (response.ok) {
                // eslint-disable-next-line no-console
                console.log('Printing...');
            } else {
                // eslint-disable-next-line no-console
                console.log('Something went wrong');
            }
        } catch (error) {
            // eslint-disable-next-line no-console
            console.log('Could not print');
        }
    };

    // Call this function from child component to update the full formState
    const onPersonalDataChange = (personalDataState: IPersonalData) => {
        setFormState({ ...formState, personalData: personalDataState });
    };

    return (
        <>
            <Form onSubmit={printForm}>
                <PersonalData defaultData={defaultData} onChange={onPersonalDataChange} />
                <Button>Skriv ut</Button>
            </Form>
        </>
    );
};

export default App;
```

If you want to use async default values for input fields that are fetched from an API, you can add another useEffect handler to listen for changes to that data. We then have to add the value attribute to our input field in order to reflect any state change that does not go through the field input directly. An example of that is implemented below:

```TSX
// PersonalData.tsx

import React, { useState } from 'react';
import { Form, Input, Select, Checkbox, Radio } from 'helium-web-ui-react';
import { IPersonalData, IDefaultData } from './AppInterface';

export interface IPersonalDataProps {
    defaultData: IDefaultData;
    onChange?: (personalData: IPersonalData) => void;
}

const PersonalData: React.FC<IPersonalDataProps> = (props: IPersonalDataProps) => {
    const { onChange, defaultData } = props;

    const [personalData, setPersonalData] = useState<IPersonalDataProps>({
        name: '',
        gender: 'Man',
        before65: false,
        after65: false,
        product: '',
    });

    useEffect(() => {
        const newState = {...personalData, name: defaultData.name};
        setPersonalData(newState);

        if(onChange) {
          onChange(newState);
        }
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [defaultData])

    const handleInputChange = (event: any) => {
        const value = event.target.type === 'checkbox' ? event.target.checked : event.target.value;
        const newState = { ...personalData, [event.target.name]: value };

        setPersonalData(newState);

        if (onChange) {
            onChange(newState);
        }
    };

    return (
        <>
            <Form.Field>
                <label htmlFor="name">Namn</label>
                <Input id="name" name="name" onChange={(e) => handleInputChange(e)} value={personalData.name}></Input>
            </Form.Field>
            <Form.Field>
                <Checkbox id="calculateBefore65" label="Beräkna före 65" name="before65" checked={personalData.before65} onChange={(e) => handleInputChange(e)} />
                <Checkbox id="calculateAfter65" label="Beräkna efter 65" name="after65" checked={personalData.after65} onChange={(e) => handleInputChange(e)} />
            </Form.Field>
            <Form.Field>
                <label htmlFor="radio">Kön</label>
                <Radio id="radio" name="gender" label="Man" value="Man" checked={personalData.gender === 'Man'} onChange={(e) => handleInputChange(e)} />
            </Form.Field>
            <Form.Field>
                <Radio id="woman" name="gender" label="Kvinna" value="Kvinna" checked={personalData.gender === 'Kvinna'} onChange={(e) => handleInputChange(e)} />
            </Form.Field>
            <Form.Field>
                <Select id="products" name="product" fieldLabel="Produkt" onChange={(e) => handleInputChange(e)}>
                    {defaultData.productList.map((product) => (
                        <option key={product}>{product}</option>
                    ))}
                </Select>
            </Form.Field>
        </>
    );
};

export default PersonalData;
```

## Async

### Await/async in React

The await/async syntax is a new way of making asynchronous requests in JS and uses promises in the background (instead of using then etc. directly). An async function is a function declared with the async keyword. Async functions are instances of the AsyncFunction constructor, and the await keyword is permitted within them. The async and await keywords enable asynchronous, promise-based behavior to be written in a cleaner style, avoiding the need to explicitly configure promise chains.

```JSX
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}

async function asyncCall() {
  console.log('calling');
  const result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: "resolved"
}

asyncCall();
```

#### Parallel fetch requests

We can wait for multiple await calls to finish like this:

```JSX
async function fetchMoviesAndCategories() {
  const [moviesResponse, categoriesResponse] = await Promise.all([
    fetch('/movies'),
    fetch('/categories')
  ]);

  const movies = await moviesResponse.json();
  const categories = await categoriesResponse.json();

  return {
    movies,
    categories
  };
}

fetchMoviesAndCategories().then(({ movies, categories }) => {
  movies;     // fetched movies
  categories; // fetched categories
});
```

### setState Asynchronous

The setState(...) method from the useState hook is **asynchronous**. This is usually not a problem but if you have multiple methods that set state after another and they all correspond to a visual change in the UI, then it could introduce race conditions.

If the new state is computed using the previous state, you can pass a function to setState. The function will receive the previous value, and return an updated value. Here’s an example of a counter component that uses both forms of setState:

```JSX
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

The ”+” and ”-” buttons use the functional form, because the updated value is based on the previous value. But the “Reset” button uses the normal form, because it always sets the count back to the initial value.

If your update function returns the exact same value as the current state, the subsequent rerender will be skipped completely.

Below is another example of how the async behaviour of setting state can cause errors on your page:

```JSX
const [counter, setCounter] = useState(0)

const handleClickBroken = () => {
  setCounter(counter + 1)
  console.log(`Counter is equal to ${counter}`) // the state will NOT have time to update!!!
}

const handleClickWorks = () => {
  const newCounter = counter + 1;
  setCounter(newCounter)
  console.log(`Counter is equal to ${newCounter}`) // the new value will be used here since we created a new local variable
}
```

In many cases the code below will work fine without adding an anonymous function call to the setState call. However, React schedules state updates, it doesn't perform them instantly and therefore if you schedule multiple state updates simultaneously, you could be depending on an incorrect/outdated state when calculating your new state.

If you use the function syntax below, React GUARANTEES that the state will be the latest snapshot, keeping all scheduled state updates in mind.

```JSX
const Component = () => {
  const [objectState, setObjectState] = useState(initialCount);

  const handleObjectStateChange = (event) => {
    setObjectState((prevState) => { // Get previous state snapshot
      return {...prevState, enteredTitle: "Test"};
    });
  }
  return (
    <>
      <button onClick={handleObjectStateChange}>Set title</button>
    </>
  );
}
```

## Lazy Initial State

The initialState argument is the state used during the initial render. In subsequent renders, it is disregarded. If the initial state is the result of an expensive computation, you may provide a function instead, which will be executed only on the initial render:

```JSX
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

## React.Fragment

A common pattern in React is for a component to return multiple elements. Fragments let you group a list of children without adding extra nodes to the DOM.

```JSX
render() {
  return (
    <React.Fragment> {/* <React.Fragment> = <> */}
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment> {/* </React.Fragment> = </> */}
  );
}
```

## Decoupling

We want our components to be re-usable in multiple places but sometimes we create them without that in mind, often because they are tightly coupled to data that we receive from an API and we just put the API call in the component file. A better way to do it is to decouple the data from the component itself.

This is the initial code we have that we want to decouple:

```JSX
import React, { useState, useEffect } from 'react';

const SomeComponent = (props) => {
  const [someData, setSomeData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch('/some-data')
      .then(response => response.json())
      .then(data => setSomeData(data))
      .catch(error => setError(error))
      .finally(() => setLoading(false));
  }, []);

  return (
    <React.Fragment>
      {loading && <div>{'Loading...'}</div>}
      {!loading && error && <div>{`Error: ${error}`}</div>}
      {!loading && !error && someData && <div>{/* INSERT SOME AMAZING UI */}</div>}
    </React.Fragment>
  );
};
```

### Custom Hooks

This is the custom hook:

```JSX
import React, { useState, useEffect } from 'react';

const useSomeData = () => {
  const cachedData = JSON.parse(localStorage.getItem('someData'));
  const [someData, setSomeData] = useState(cachedData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!someData) {
      setLoading(true);
      fetch('/some-data')
        .then(response => response.json())
        .then(data => {
          localStorage.setItem('someData', JSON.stringify(data));
          return setSomeData(data);
        })
        .catch(error => setError(error))
        .finally(() => setLoading(false));
    }
  }, []);

  return { someData, loading, error };
};
```

New component now looks like this:

```JSX
const SomeComponent = (props) => {
  const { someData, loading, error } = useSomeData();
  return (
    <React.Fragment>
      {loading && <div>{'Loading...'}</div>}
      {!loading && error && <div>{`Error: ${error}`}</div>}
      {!loading && !error && someData && <div>{/* INSERT SOME AMAZING UI */}</div>}
    </React.Fragment>
  );
};

const AnotherComponent = (props) => {
  const { someData, loading, error } = useSomeData();
  return (
    <React.Fragment>
      {loading && <div>{'Loading...'}</div>}
      {!loading && error && <div>{`Error: ${error}`}</div>}
      {!loading && !error && someData && <div>{/* INSERT ANOTHER AMAZING UI */}</div>}
    </React.Fragment>
  );
};
```

### Render props

An alternative approach is using render props. Instead of a custom hook that returns someData, loading, and error, let’s create a Render Props component — SomeData — that wraps around a function (i.e., children needs to be a function), implements the data logic, and passes in someData, loading, and error into the function.

```JSX
import React, { useState, useEffect } from 'react';

const SomeData = ({ children }) => {
  // DATA FETCHING/MANAGEMENT FRAMEWORK OR LIBRARY OF YOUR CHOICE
  return children({ someData, loading, error });
};

const SomeComponent = ({ someData, loading, error }) => (
  <React.Fragment>
    {loading && <div>{'Loading...'}</div>}
    {!loading && error && <div>{`Error: ${error}`}</div>}
    {!loading && !error && someData && <div>{/* INSERT SOME AMAZING UI */}</div>}
  </React.Fragment>
);

const SomeComponentWithSomeData = () => (
  <SomeData>
    {renderProps => (<SomeComponent {...renderProps} />)}
  </SomeData>
);
```

You can replace line 4 in the snippet above with Redux, Apollo GraphQL, or any data fetching/management layer of your choice.

We can now reuse SomeComponent (UI component) without SomeData (Render Props component). We can also reuse SomeData without SomeComponent.

### Higher Order Components (HOCs)

A third approach would be using HOCs. Let’s create a HOC — withSomeData — that accepts a React component as an argument, implements the data logic, and passes someData, loading, and error as props into the wrapped React component.

```JSX

import React, { useState, useEffect } from 'react';

const withSomeData = Component => {
  const ComponentWithSomeData = (props) => {
    // DATA FETCHING/MANAGEMENT FRAMEWORK OR LIBRARY OF YOUR CHOICE
    return <Component {...props} someData={someData} loading={loading} error={error} />;
  };
  return ComponentWithSomeData;
};

const SomeComponent = ({ someData, loading, error }) => (
  <React.Fragment>
    {loading && <div>{'Loading...'}</div>}
    {!loading && error && <div>{`Error: ${error}`}</div>}
    {!loading && !error && someData && <div>{/* INSERT SOME AMAZING UI */}</div>}
  </React.Fragment>
);

const SomeComponentWithSomeData = withSomeData(SomeComponent);
```

You can replace line 5 in the snippet above with Redux, Apollo GraphQL, or any data fetching/management layer of your choice.

We can now reuse SomeComponent (UI component) without withSomeData (HOC). We can also reuse withSomeData without SomeComponent.

## The "key" value

When you render a list of components in React you have to specify a key value, why? That is because React wants to render your application as efficiently as possible so it wants to know when to re-render elements. It is NOT advised to use the index of the list because it might have detrimental effects on performance, at least if the list is dynamic. If you delete or add an element to the list, EVERY item will get re-rendered because every id will effectively change.

> TL;DR: Specify unique id:s (keys) for items in a list and DO NOT use indices.

## Controlled Components

In React, most components are controlled components which basically means that the components are tied to data stored in a React state. Tha value of the state is bound to inputs for instance. If we have an input field with an onChange handler that does not do anything, you won't be able to type in that field. That is because we need to access the event.target.value field and update the state to modify the input field value.

Below is an example of a controlled component where the state controls the value of the input field:

```JSX
import React, { useState } from "react";

export default function App() {
  const [inputValue, setInputValue] = useState("");
  const handleInputChange = (e) => {
    setInputValue(e.target.value)
  }
  const handleSubmitButton = () => {
    alert(inputValue);
  };
  return (
    <div className="App">
      <input value={inputValue} onChange={handleInputChange} />
      <input type="submit" value="submit" onClick={handleSubmitButton} />
    </div>
  );
}
```

in the above example, we use the controlled component to handle the form input value using React Hooks and every time you will type a new character, handleInputChange is called and it takes in the new value of the input and sets it in the state. By having "value={inputValue}" in the code, we have two-way data binding and bind the react state back to the value property of the input field.

The uncontrolled component is like traditional HTML form inputs that you will not be able to handle the value by yourself but the DOM will take care of handling the value of the input and save it then you can get this value using React Ref and for example, print it inside alert when submitting or play with this value as you want.

Here is an example of an uncontrolled component:

```JSX
import React, { useRef } from "react";

export default function App() {
  const inputRef = useRef(null);
  const handleSubmitButton = () => {
    alert(inputRef.current.value);
  };
  return (
    <div className="App">
      <input type="text" ref={inputRef} />
      <input type="submit" value="submit" onClick={handleSubmitButton} />
    </div>
  );
}
```

> _defaultValue_ is an attribute that is typically used for uncontrolled components to set a default initial value for an input. If you want to set initial values for controlled components, use the _value_ attribute.

## Dynamic imports

Instead of downloading the entire app before users can use it, code splitting allows you to split your code into small chunks which you can then load on demand.

This project setup supports code splitting via dynamic import(). Its proposal is in stage 4. The import() function-like form takes the module name as an argument and returns a Promise which always resolves to the namespace object of the module.

Here is an example:

moduleA.js

```JSX
const moduleA = 'Hello';

export { moduleA };
```

App.js

```JSX
import React, { Component } from 'react';

class App extends Component {
  handleClick = () => {
    import('./moduleA')
      .then(({ moduleA }) => {
        // Use moduleA
      })
      .catch(err => {
        // Handle failure
      });
  };

  render() {
    return (
      <div>
        <button onClick={this.handleClick}>Load</button>
      </div>
    );
  }
}

export default App;
```

This will make `moduleA.js` and all its unique dependencies as a separate chunk that only loads after the user clicks the 'Load' button. For more information on the chunks that are created, see the production build section.

You can also use it with `async / await` syntax if you prefer it.

## React.lazy

React.lazy was added in React 16.6 and is a great feature to split bundles that are created for your website. Code-splitting your app can help you “lazy-load” just the things that are currently needed by the user, which can dramatically improve the performance of your app. While you haven’t reduced the overall amount of code in your app, you’ve avoided loading code that the user may never need, and reduced the amount of code needed during the initial load.

With React.lazy we can tell React to load the page as usual but show a loading icon (or whatever we want) for a specific component that takes some time to load.

Here is an example:

```JSX
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

## React.memo

Wrapping a component with React.memo causes it to only re-render when the props change.
By default, react will always rerender the component if the parent is changed so React.memo can be very useful when a child is forced to re-render.

Below is some example code using React.memo. This component will never be re-rendered unless title or releaseDate changes.

```JSX
export function Movie({ title, releaseDate }) {
  return (
    <div>
      <div>Movie title: {title}</div>
      <div>Release date: {releaseDate}</div>
    </div>
  );
}

export const MemoizedMovie = React.memo(Movie);
```

We can also use a custom equality resolver to define what "equality" should mean:

```JSX
function moviePropsAreEqual(prevMovie, nextMovie) {
  return prevMovie.title === nextMovie.title
    && prevMovie.releaseDate === nextMovie.releaseDate;
}

const MemoizedMovie2 = React.memo(Movie, moviePropsAreEqual);
```

## useCallback

useCallback prevents react from always recreating functions. Assume we have code like below:

```JSX
const [count, setCount] = useState(0);

const increment = useCallback(() => {
    setCount(c => c + 1);
}, [setCount]);
```

In this case, whenever count or setCount changes the function increment will be recreated. This means that we do not recreate the setCount function every time the count variable changes.

useCallback is often used in these two scenarios:

1. When we want to prevent functions from changing the value. Often combined with `React.memo({ increment })` or something similar. If we in this case pass in an increment function that gets recreated (and thus rerendered) every time we use it, then React.memo does nothing for us. If we make use of useCallback as in the example above and pass in that function, the increment parameter will not change on every render and React.memo will then optimize.

2. If we use useEffect and have some logic and depend on the increment function, then we don't want useEffect to fire off all the time. In this case we can make use of useCallback as well.

## useRef

Sometimes we want to interact with the underlying DOM directly and not the virtual DOM. The useRef hook and the ref attribute gives us a way to do that.

```JSX
const inputRef = useRef();

<input ref={inputRef} name="email" value="Test" onChange={handleChange}>

<button onClick={() => inputRef.current.focus()}>Focus</button>
```

```JSX
const renders = useRef(0);

console.log("Hello renders: ", renders.current++);
```

useRef use cases:

1. If we want a reference to a component, for instance if we want to focus a textfield whenever we click a button we can store a reference to the textfield.
2. Store the number of times a component has rendered. Incrementing the ref does not cause a rerender so we can use it to check how many times a component renders.

## useContext

Accepts a context object (the value returned from React.createContext) and returns the current context value for that context. The current context value is determined by the value prop of the nearest <MyContext.Provider> above the calling component in the tree.

When the nearest <MyContext.Provider> above the component updates, this Hook will trigger a rerender with the latest context value passed to that MyContext provider. Even if an ancestor uses React.memo or shouldComponentUpdate, a rerender will still happen starting at the component itself using useContext.

> A component calling useContext will always re-render when the context value changes. If re-rendering the component is expensive, you can optimize it by using memoization.

Below is an example of using the useContext hook for a ThemeContext:

```JSX
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

## useReducer

An alternative to useState. Accepts a reducer of type (state, action) => newState, and returns the current state paired with a dispatch method. (If you’re familiar with Redux, you already know how this works.)

useReducer is usually preferable to useState when you have complex state logic that involves multiple sub-values or when the next state depends on the previous one. useReducer also lets you optimize performance for components that trigger deep updates because you can pass dispatch down instead of callbacks.

Here’s a counter example, using a reducer instead of useState:

```JSX
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

## useMemo

Returns a memoized value.

Pass a “create” function and an array of dependencies. useMemo will only recompute the memoized value when one of the dependencies has changed. This optimization helps to avoid expensive calculations on every render.

Remember that the function passed to useMemo runs during rendering. Don’t do anything there that you wouldn’t normally do while rendering. For example, side effects belong in useEffect, not useMemo.

If no array is provided, a new value will be computed on every render.

You may rely on useMemo as a performance optimization, not as a semantic guarantee. In the future, React may choose to “forget” some previously memoized values and recalculate them on next render, e.g. to free memory for offscreen components. Write your code so that it still works without useMemo — and then add it to optimize performance.

```JSX
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

## Repeatable components

Sometimes you want to be able to dynamically create the same block of code over and over. You can do this by using names with index suffixes in your state.

The interfaces:

```TSX
// AppInterface.tsx
export interface IRepeatableComponentWrapper: {
  tmp1: string;
  tmp2: string;
  repeatableComponents: { [key: string] : IRepeatableComponent }
}

export interface IRepeatableComponent {
  test?: string;
}
```

The repeatable component content (RepeatableComponent.tsx):

```TSX
import React, { useState } from 'react';
import { Form, Input } from 'semantic-ui-react';
import { IRepeatableComponent } from './AppInterface';

interface IRepeatableComponentProps {
  id: string;
  onChange?: (repeatableComponent: IRepeatableComponent, index: string) => void;
}

const RepeatableComponent = ({id, onChange}: IRepeatableComponentProps) = {...}
export default RepeatableComponent;
```

Repeatable component (MultipleCollapseBlock.tsx):

```TSX
import * as React from 'react';
import { Button, Collapse, Form, Icon, Segment, ButtonProps } from ‘semantic-ui-react’;

export interface IMultipleCollapseBlockProps {
    [prop: string]: any;
    name: string;
    ctaExpand: string;
    ctaRemove: string;
    ctaAdd: string;
    max?: number;
    onClickAddHandler?: (event: React.MouseEvent<HTMLButtonElement>, data: ButtonProps) => void;
    onClickRemoveHandler?: (event: React.MouseEvent<HTMLButtonElement>, data: ButtonProps) => void;
}

export const MultipleCollapseBlock = ({ name, ctaExpand, ctaRemove, ctaAdd, onClickRemoveHandler, onClickAddHandler, children, max = 5 }: React.PropsWithChildren<IMultipleCollapseBlockProps>) => {
    const isOpen = React.useCallback(() => {
        return React.Children.count(children) > 0;
    }, [children]);

    const showAddMore = React.useCallback(() => {
        return React.Children.count(children) < max;
    }, [children, max]);

    return (
        <Form.Field>
            {!isOpen() && (
                <>
                    <Button renderAs="pill" secondary onClick={onClickAddHandler} className="button--clean">
                        <Icon name="add circle" />
                        {ctaExpand}
                    </Button>
                </>
            )}

            {isOpen() &&
                React.Children.map(children, (child) => {
                    return (
                        <div key={(child as React.Component<any>).props.id}>
                            <div className="label-wrapper">
                                <Button renderAs="pill" secondary onClick={onClickRemoveHandler} className="button--clean button--right" data-field-id={(child as React.Component<any>).props.id}>
                                    <Icon name="remove circle" />
                                    {ctaRemove}
                                </Button>
                            </div>
                            <Collapse isOpened={isOpen()} collapseSpeed="normal" className="multiple-collapse--margin-bottom">
                                <Segment secondary>{child}</Segment>
                            </Collapse>
                        </div>
                    );
                })}

            {isOpen() && showAddMore() && (
                <Button renderAs="pill" secondary onClick={onClickAddHandler} className="button--clean">
                    <Icon name="add circle" />
                    {ctaAdd}
                </Button>
            )}
        </Form.Field>
    );
};
```

> Collapse is a component that animates the opening and closing of its children.

The parent of the repeatable component:

```TSX
import { ButtonProps } from 'semantic-ui-react';

const [repeatableComponentWrapperState, setRepeatableComponentWrapperState] = useState();
const [repeatableComponentIndex, setRepeatableComponentIndex] = React.useState<number>(0);
const repeatableComponentStateName = 'RepeatableComponent';

// Filter out any repeatable components
const findRepeatableComponents = () => {
  const result = Object.keys(repeatableComponentWrapperState.repeatableComponents).filter((key: string) => key.startsWith(`${repeatableComponentStateName}_`));
  return result;
}

// Handle changes to repeatable components
const onRepeatableComponentChange = (repeatableComponent: IRepeatableComponent, index: string) => {
  const newState = { ...repeatableComponentWrapperState, { ...repeatableComponentWrapperState.repeatableComponents, [index]: repeatableComponent } };
  setRepeatableComponentWrapperState(newState);
}

const handleRepeatableComponentRemove = (event: React.MouseEvent<HTMLButtonElement>, data: ButtonProps) => {
  event.preventDefault();
  // Id to remove must be stored as an attribute on the element
  const fieldToRemove = data['data-field-id'];
  // Destructure object and take out "the rest"
  const { [`${fieldToRemove}`]: _, ...stateWithoutFieldToRemove } = repeatableComponentWrapperState.repeatableComponents;
  const newState = { ...repeatableComponentWrapperState,  repeatableComponents: stateWithoutFieldToRemove};

  setRepeatableComponentWrapperState(newState);
}

const handleRepeatableComponentAdd = (event: React.MouseEvent<HTMLButtonElement>) => {
  event.preventDefault();
  // Id to remove must be stored as an attribute on the element
  const newState = { ...repeatableComponentWrapperState,  repeatableComponents: { ...repeatableComponentWrapperState.repeatableComponents, [`${repeatableComponentStateName}`]: {} }};
  setRepeatableComponentWrapperState(newState);
  setRepeatableComponentIndex(repeatableComponentIndex +  1);

return (
  <MultipleCollapseBlock max={10} name="test" ctaAdd="Lägg till fler"
ctaRemove="Ta bort" ctaExpand="Lägg till" onClickAddHandler={handleRepeatableComponentAdd} onClickRemoveHandler={handleRepeatableComponentRemove}>
    {findRepeatableComponents().map((key) => (
        <RepeatableComponent key={key} id={key} onChange={onRepeatableComponentChange} />
    ))}
  </MultipleCollapseBlock>)
}
```
