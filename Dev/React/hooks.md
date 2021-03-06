## Hooks

- [useState](#useState)
- [useCallback](#useCallback)
- [useEffect](#useEffect)
- [Custom Hook example](#custom-hook-example)
- [Tips & Best practices](#Tips-n-best-practice)

### useState

When there is too many states to be managed, you can do this:

```js
const initialState = { loading: false, results: [], value: '' };

const App = () => {
  const [{ loading, results, value }, setState] = useState(initialState);

  // setState
  setState(prevState => ({ ...prevState, loading: true, value }));

  // reset all states
  setState(prevState => ({ ...initialState }));
};
```

### useCallback

For better performance. i.e avoid creating new func repeatedly which causes unnecessary re-rendering of child component which accepts the callback as a prop. Put deps in the array as the 2nd argument, when you access them from inside the func.

```js
const { setNav } = props;
const [value, setValue] = useState('');

const onClick = useCallback(
  e => {
    setNav(value); // put value in dep array as you access it!
  },
  [setNav, value]
);
```

Note, it's a perfect fit for `useCallback` in the above example as `value` referenced by arrow function is defined outside the arrow function scope.

### useEffect

Do something only once:

```js
useEffect(() => {
  api.fetch('/data');
}, []); // empty array means no dependency to watch on
```

See how you can refactor lifecycle hooks based code to `useEffect` one.

```js
// old
import React, { Component } from 'react';
import websockets from 'websockets';

class ChatChannel extends Component {
  state = {
    messages: [];
  }

  componentDidMount() {
    this.startListeningToChannel(this.props.channelId);
  }

  componentDidUpdate(prevProps) {
    if (this.props.channelId !== prevProps.channelId) {
      this.stopListeningToChannel(prevProps.channelId);
      this.startListeningToChannel(this.props.channelId);
    }
  }

  componentWillUnmount() {
    this.stopListeningToChannel(this.props.channelId);
  }

  startListeningToChannel(channelId) {
    websockets.listen(
      `channels.${channelId}`,
      message => {
        this.setState(state => {
          return { messages: [...state.messages, message] };
        });
      }
    );
  }

  stopListeningToChannel(channelId) {
    websockets.unlisten(`channels.${channelId}`);
  }

  render() {
    // ...
  }
}


// new
import React, { useEffect, useState } from 'react';
import websockets from 'websockets';

function ChatChannel({ channelId }) {
  const [messages, setMessages] = useState([]);

  // cleanup callback will be triggered either when channelId changes, or when the component unmounts.
  useEffect(() => {
    websockets.listen(
      `channels.${channelId}`,
      message => setMessages(messages => [...messages, message])
    );

    return () => websockets.unlisten(`channels.${channelId}`);
  }, [channelId]);

  // ...
}
```

> Instead of thinking about when we should apply the side effect, we declare the side effect’s dependencies. This way React knows when it needs to run, update, or clean up.
> That’s where the power of useEffect lies. The websocket listener doesn’t care about mounting and unmounting components, it cares about the value of channelId over time.

### Custom Hook Example
Write custom hook when you feel a need to share common logic.

```jsx
// hook
function useSmartPassword() {
  const [isValid, setValid] = useState(false);

  const onChange = e => {
    const newValue = e.target.value;
    let _isValid = false;
    if (newValue.length >= 8) _isValid = true;
    setValid(_isValid);
  };

  return [isValid, onChange];
}

function Form() {
  const [isValid, onPasswordChange] = useSmartPassword();
  return (
    <div className="Form">
      <label htmlFor="password">Password: </label>
      <input id="password" onChange={e => onPasswordChange(e)} />
      {isValid ? <p>Your password is valid </p> : null}
    </div>
  );
}
```

### Tips-n-best-practice

- Declare functions needed by an effect inside of it:

```js
// bad
function Example({ someProp }) {
  function doSomething() {
    console.log(someProp);
  }

  useEffect(() => {
    doSomething();
  }, [someProp]);
}

// good
function Example({ someProp }) {
  useEffect(() => {
    function doSomething() {
      console.log(someProp);
    }

    doSomething();
  }, [someProp]);
}
```

- React uses `Object.is(a, b)` to do referential equality check.
- The function passed to `useEffect` will run after the render is committed to the screen.
