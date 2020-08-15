# Section 1: Getting Started

## What is React?
React is a JavaScript Library for building User Interfaces. React apps run in the browser, not in the server. Thus, they are very reactive and the users do not have to wait for the server when rendering.

On a React page you would split a page into components which you can then reuse throughout the application, such as Header, Sidebar, Headline, ArticleContent etc. The components can be thought of as custom Html elements.

React uses JSX which is just syntactic sugar for normal JavaScript code.

## How does React work?
At the very root of your application, there is typically a line like this or something similar: 

```JSX
ReactDOM.render(<App/>, document.getElementById('root'));
```

This line uses React DOM, which is how React renders javascript functions as a component to the real DOM. Basically, in our `index.html` we have a div with an id of "root" which is the location in our DOM where all our React components are rendered.

The thing that makes React components so reusable is that we can pass properties or props to them. These props can be either values or functions.

```JSX
function Person(props) {
    return (
        <div className="person">
            <h1>{props.name}</h1>
            <p>Your age: {props.age}</p>
        </div>
    );
}

ReactDOM.render(<Person name="Jacob" age="25" />, document.querySelector('#person'));

// index.html
<div id="person"></div>
```

The example above shows how we could render a Person component and pass in values for the name and the age.

## Why React?

## Single Page vs Multi Page Applications

## React Alternatives