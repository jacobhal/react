# React - The Complete Guide (incl Hooks, React Router, Redux)
This is a repository for the Udemy course "React - The Complete Guide (incl Hooks, React Router, Redux)". This repository README contains general tips and notes that I like to keep in mind. If you navigate to the folders you will see the READMEs of each section of the course. The outer level also contains the source code for the projects that were created in this course.

## Certificate
TODO

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

in the above example, we use the controlled component to handle the form input value using React Hooks and every time you will type a new character, handleInputChange is called and it takes in the new value of the input and sets it in the state then you can use this value and print it inside alert when submitting use handleSubmitButton

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
