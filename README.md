# React

## React.memo
Wrapping a component with React.memo causes it to only re-render when the props change.
By default, react will always rerender the component if the parent is changed.

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