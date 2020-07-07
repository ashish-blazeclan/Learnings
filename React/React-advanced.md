## CODE SPLITTING
The best way to introduce code-splitting into your app is through the dynamic import() syntax.
```javascript
import { add } from './math';

console.log(add(16, 26));
```
#### Lazy loading
```javascript
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
------------------------------------------------------------------------------
## CONTEXT
Context provides a way to pass data through the component tree without having to pass props down manually at every level.
```javascript
// Context lets us pass a value deep into the component tree
// without explicitly threading it through every component.
// Create a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to
// pass the theme down explicitly anymore.
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Assign a contextType to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```
If you only want to avoid passing some props through many levels, component composition is often a simpler solution than context.
Common examples where using context might be simpler than the alternatives include managing the current locale, theme, or a data cache

------------------------------------------------------------------------------
## ERRRO BOUNDARIES
Error boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.

Error boundaries do not catch errors for:

1. Event handlers (learn more)
2. Asynchronous code (e.g. setTimeout or requestAnimationFrame callbacks)
3. Server side rendering
4. Errors thrown in the error boundary itself (rather than its children)
   If you need to catch an error inside event handler, use the regular JavaScript try / catch statement:
   Then why not use try-catch instead of error boundaries?
   try-catch is imperative and error-boundaries are declaritive, since react is declarative we use...
A class component becomes an error boundary if it defines either (or both) of the lifecycle methods static getDerivedStateFromError() or componentDidCatch(). Use static getDerivedStateFromError() to render a fallback UI after an error has been thrown. Use componentDidCatch() to log error information.
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // You can also log the error to an error reporting service
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```
Only class components can be error boundaries.
As of React 16, errors that were not caught by any error boundary will result in unmounting of the whole React component tree.

------------------------------------------------------------------------------
## REFS and the DOM
Refs provide a way to access DOM nodes or React elements created in the render method.
In the typical React dataflow, props are the only way that parent components interact with their children. To modify a child, you re-render it with new props. However, there are a few cases where you need to imperatively modify a child outside of the typical dataflow e.g. Managing focus, text selection, or media playback.

Avoid using refs for anything that can be done declaratively.

For example, instead of exposing open() and close() methods on a Dialog component, pass an isOpen prop to it.

Don’t Overuse Refs.

#### Creating refs
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}

//Accessing refs
const node = this.myRef.current;

//adding refs to class component
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
// Note that this only works if CustomTextInput is declared as a class
```
If you want to allow people to take a ref to your function component, you can use forwardRef.
You can, however, use the ref attribute inside a function component as long as you refer to a DOM element or a class component:

------------------------------------------------------------------------------
## Fragments
Fragments let you group a list of children without adding extra nodes to the DOM.
```javascript
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}

// Use short syntax instead, to do replace 
<React.Fragment>...</React.Fragment>
with
<>...</>
```

------------------------------------------------------------------------------
## Higher-Order Components
A higher-order component (HOC) is an advanced technique in React for reusing component logic.
A higher-order component is a function that takes a component and returns a new component.
```javascript
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

HOCs are used for Cross-Cutting Concerns
**Concerns-** term that refers to a part of the system divided on the basis of the functionality e.g. business logic is a concern
**Cross-Cutting Concerns-** a concern which is applicable throughout the application and it affects the entire application.
e.g. logging, security

 e.g. use
```javascript
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```
Imagine we have exactly same component 'blogpost' with same pattern, just the datasorce is different, But much of their implementation is the same:

1. On mount, add a change listener to DataSource.
2. Inside the listener, call setState whenever the data source changes.
3. On unmount, remove the change listener.

You can imagine that in a large app, this same pattern of subscribing to DataSource and calling setState will occur over and over again. We want an abstraction that allows us to define this logic in a single place and share it across many components. This is where higher-order components excel.

We can write a function that creates components, like CommentList and BlogPost, that subscribe to DataSource. The function will accept as one of its arguments a child component that receives the subscribed data as a prop. Let’s call the function withSubscription

```javascript
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource) => DataSource.getBlogPost()
);


// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```
A HOC is a pure function with zero side-effects.

------------------------------------------------------------------------------
## JSX in depth
```javascript
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>

//compiles to 
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
// thus react must be in scope i.e, import React from 'react'
```
User-Defined Components Must Be Capitalized.
#### Children in JSX
```javascript
<MyComponent>Hello world!</MyComponent>
```
This is valid JSX, and props.children in MyComponent will simply be the string "Hello world!". 

------------------------------------------------------------------------------
## Optimizing performance
1. For the most efficient Brunch production build, install the terser-brunch plugin:
Then, to create a production build, add the -p flag to the build command:
brunch build -p

#### 2. Avoid Reconciliation

Process - 
When a component’s props or state change, React renders the element and then decides whether an actual DOM update is necessary by comparing the newly returned element with the previously rendered one. When they are not equal, React will update the DOM.

You can even avoid re-rendering of a acomponent.
If you know that in some situations your component doesn’t need to update, you can avoid re-rendering by overriding the lifecycle function shouldComponentUpdate, which is triggered before the re-rendering process starts.

```javascript
shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }
```
In most cases, instead of writing shouldComponentUpdate() by hand, you can inherit from React.PureComponent. It is equivalent to implementing shouldComponentUpdate() with a shallow comparison of current and previous props and state.

## 3. Use Profiler API
The Profiler measures how often a React application renders and what the “cost” of rendering is. Its purpose is to help identify parts of an application that are slow and may benefit from optimizations such as memoization.
Profiling adds some additional overhead, so it is disabled in the production build.

It requires two props: an id (string) and an onRender callback (function) which React calls any time a component within the tree “commits” an update.
```javascript
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);
```
onRender callback
```javascript
function onRenderCallback(
  id, // the "id" prop of the Profiler tree that has just committed
  phase, // either "mount" (if the tree just mounted) or "update" (if it re-rendered)
  actualDuration, // time spent rendering the committed update
  baseDuration, // estimated time to render the entire subtree without memoization
  startTime, // when React began rendering this update
  commitTime, // when React committed this update
  interactions // the Set of interactions belonging to this update
) {
  // Aggregate or log render timings...
}
```

## 4. Use keys
Inserting an element at the beginning has worse performance. For example, converting between these two trees works poorly
```javascript
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```
React will mutate every child instead of realizing it can keep the **Duke** and **Villanova** subtrees intact. This inefficiency can be a problem.

In order to solve this issue, React supports a key attribute. When children have keys, React uses the key to match children in the original tree with children in the subsequent tree.
```javascript
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```
Keys should be stable, predictable, and unique. Unstable keys (like those produced by Math.random()) will cause problems



------------------------------------------------------------------------------
## Portals
Portals provide a first-class way to render children into a DOM node that exists outside the DOM hierarchy of the parent component.
```javascript
ReactDOM.createPortal(child, container)

//The first argument (child) is any renderable React child, such as an element, string, or fragment.
//The second argument (container) is a DOM element.
```

#### Usage
A typical use case for portals is when a parent component has an overflow: hidden or z-index style, but you need the child to visually “break out” of its container. For example, dialogs, hovercards, and tooltips.

------------------------------------------------------------------------------
## Render Props
The term “render prop” refers to a technique for sharing code between React components using a prop whose value is a function.

A component with a render prop takes a function that returns a React element and calls it instead of implementing its own render logic.
```javascript
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```
#### Use Render Props for Cross-Cutting Concerns
For example, the following component tracks the mouse position in a web app:
```javascript
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        <h1>Move the mouse around!</h1>
        {/* ...but how do we render something other than a <p>? */}
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}

// it's parent component
<MouseTracker />

// if we want to use same component functionality but that component should render something else.
// for e.g. above, instead of rendering <p> other component wants to render <div>
// this is not achievable by calling <MouseTracker> component multiple times
// we can use render props instead
```

Using Render props
```javascript
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        <h1>Move the mouse around!</h1>
        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
 
      </div>
    );
  }
}

// it's parent component
<MouseTracker render={(mouse) => (
         <div>The current mouse position is ({mouse.x}, {mouse.y})</div>
        )}/>

// Check if the code is working
```
------------------------------------------------------------------------------
## Static Type Checking
Static type checkers like Flow and TypeScript identify certain types of problems before you even run your code. They can also improve developer workflow by adding features like auto-completion. For this reason, we recommend using Flow or TypeScript instead of PropTypes for larger code bases.

Typescript/Flow can be added to react app
**npx create-react-app my-app --template typescript**

------------------------------------------------------------------------------
## Strict Mode
StrictMode is a tool for highlighting potential problems in an application. Like Fragment, StrictMode does not render any visible UI. It activates additional checks and warnings for its descendants.

You can enable strict mode for any part of your application. For example:
```javascript
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```
StrictMode currently helps with:

1. Identifying components with unsafe lifecycles - certain legacy lifecycle methods are unsafe for use in async React applications
2. Warning about legacy string ref API usage
3. Warning about deprecated findDOMNode usage
4. Detecting unexpected side effects **
5. Detecting legacy context API

------------------------------------------------------------------------------
## Proptypes
React has some built-in typechecking abilities i.e, proptypes.
Use them for small projects, got big projects, use Flow or Typescript
```javascript
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};

// if type of name is not string, warning will be shown on console.

//few more validators
optionalArray: PropTypes.array,
optionalBool: PropTypes.bool,
optionalFunc: PropTypes.func,
optionalNumber: PropTypes.number,
optionalObject: PropTypes.object,
optionalString: PropTypes.string,
optionalSymbol: PropTypes.symbol,

// custom proptypes can also be created
// default values can also be passed to props
Greeting.defaultProps = {
  name: 'Stranger'
};
```

------------------------------------------------------------------------------
## Uncontrolled components
When form data is handled by the DOM itself, and not by React

instead of writing an event handler for every state update, you can use a ref to get form values from the DOM
```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}

// if you forgot, below is the controlled component
<input value={someValue} onChange={handleChange} />
```
You can also make a controlled component as uncontrolled by removing event handler and adding ref. (in above code, I have made input tag both controlled and uncontrolled)
But this is not recommended.
File input tag is always uncontrolled component, as only DOM (user) can control it's value.

------------------------------------------------------------------------------
## Web Components
Nai aata kuch :stuck_out_tongue: