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
* UI State becomes difficult to manage with vanilla JavaScript.
* Focus on business logic, not on preventing your App from exploding.
* React has a big development team, so chances are that their code is much better than you could do yourself.
* Huge Ecosystem, active community, high performance.

## React Alternatives
* Angular
* Vue

## Single Page vs Multi Page Applications
There are essentially two kinds of web applications that can be built with frameworks such as React, Angular and Vue: Single Page Applications or Multi Page Applications.

### Single Page Application
* Only ONE HTML page, content is (re)rendered on client.
* Entire page is under React's control and we get a good user experience.
* React.render() will be called one time.

### Multi Page Application
* Multiple HTML pages, content is rendered on the server. We move between URLs or routes.
* Each page can be rendered by React with multiple React components but to move between HTML pages we have to request it from the server.
* React.render() will be called multiple times.