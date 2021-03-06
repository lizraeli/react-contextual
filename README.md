[![Build Status](https://travis-ci.org/drcmda/react-contextual.svg?branch=master)](https://travis-ci.org/drcmda/react-contextual) [![codecov](https://codecov.io/gh/drcmda/react-contextual/branch/master/graph/badge.svg)](https://codecov.io/gh/drcmda/react-contextual) [![npm version](https://badge.fury.io/js/react-contextual.svg)](https://badge.fury.io/js/react-contextual)

    npm install react-contextual

# Why 🤔

* consume (and create) context with ease, every kind of context, no matter which or whose or how many providers
* a cooked down redux-like store pattern with setState semantics and central actions

Click [this link](https://github.com/drcmda/react-contextual/blob/master/PITFALLS.md) for a detailed explanation.

# If you just need a light-weight no-frills store 🎰

<b>Examples</b>: [Counter](https://codesandbox.io/embed/3vo9164z25) | [Global setState](https://codesandbox.io/embed/01l8z634qn) | [Async actions](https://codesandbox.io/embed/lxly45lvkl) | [Memoization/Reselect](https://codesandbox.io/embed/yvx9my007z) | [Multiple stores](https://codesandbox.io/embed/0o8pj1jz7v) | [External store](https://codesandbox.io/embed/jzwv46729y)

Use [Provider](https://github.com/drcmda/react-contextual/blob/master/API.md#provider) to distribute state and actions. Connect components either by using a [HOC](https://github.com/drcmda/react-contextual/blob/master/API.md#subscribe) or [render-props](https://github.com/drcmda/react-contextual/blob/master/API.md#subscribe-as-a-component).

#### Render props

```jsx
import { Provider, Subscribe } from 'react-contextual'

const store = {
    initialState: { count: 0 },
    actions: {
        up: () => state => ({ count: state.count + 1 }),
        down: () => state => ({ count: state.count - 1 }),
    },
}

const App = () => (
    <Provider {...store}>
        <Subscribe>
            {props => (
                <div>
                    <h1>{props.count}</h1>
                    <button onClick={props.actions.up}>Up</button>
                    <button onClick={props.actions.down}>Down</button>
                </div>
            )}
        </Subscribe>
    </Provider>,
)
```

#### Higher Order Component

```jsx
import { Provider, subscribe } from 'react-contextual'

const View = subscribe()(props => (
    <div>
        <h1>{props.count}</h1>
        <button onClick={props.actions.up}>Up</button>
        <button onClick={props.actions.down}>Down</button>
    </div>
))

const App = () => (
    <Provider {...store}>
        <View />
    </Provider>,
)
```

#### With decorator

```jsx
@subscribe()
class View extends React.PureComponent {
    // ...
}
```

#### External store

If you prefer, maintain your own store via [createStore](https://github.com/drcmda/react-contextual/blob/master/API.md#createstore). It is fully reactive and features a basic subscription model, similar to a redux store. You can use it as reference for consumers as well. 

```jsx
import { Provider, createStore, subscribe } from 'react-contextual'

const externalStore = createStore({
    initialState: { count: 0 },
    actions: { up: () => state => ({ count: state.count + 1 }) },
})

const Test = subscribe(externalStore)(
    props => <button onClick={() => props.actions.up()}>{props.count}</button>,
)

const App = () => (
    <Provider store={externalStore}>
        <Test />
    </Provider>,
)
```

#### Global setState

If you do not supply actions [createStore](https://github.com/drcmda/react-contextual/blob/master/API.md#createstore) will add setState by default. This applies to both createStore and the Provider above.

```jsx
const store = createStore({ initialState: { count: 0 } })

const Test = subscribe(store)(
    props => (
        <button onClick={() => props.actions.setState(state => ({ count: state.count + 1 }))}>
            {props.count}
        </button>
    ),
)
```

#### mapContextToProps

[subscribe](https://github.com/drcmda/react-contextual/blob/master/API.md#subscribe) and [Subscribe](https://github.com/drcmda/react-contextual/blob/master/API.md#subscribe-as-a-component) (the component) work with any React context, even polyfills. They pick providers and select state. Extend wrapped components from `React.PureComponent` and they will only render when picked state has changed.

```jsx
// Subscribes to all contents of the provider
subscribe(context)
// Picking a variable from the store, the component will only render when it changes ...
subscribe(context, store => ({ loggedIn: store.loggedIn }))
// Picking a variable from the store using the components own props
subscribe(context, (store, props) => ({ user: store.users[props.id] }))
// Making store context available under the 'store' prop
subscribe(context, 'store')
// Selecting several providers
subscribe([Theme, Store], (theme, store) => ({ theme, store }))
// Selecting several providers using the components own props
subscribe([Theme, Store], (theme, store, props) => ({ store, theme: theme.colors[props.id] }))
// Making two providers available under the props 'theme' and 'store'
subscribe([Theme, Store], ['theme', 'store'])
```

# If you like to provide context 🚀

<b>Examples</b>: [Global context](https://codesandbox.io/embed/v8pn13nq77) | [Transforms](https://codesandbox.io/embed/mjv84k1kn9) | [Unique context](https://codesandbox.io/embed/ox405qqopy) | [Imperative context](https://codesandbox.io/embed/30ql1rxzlq) | [Generic React Context](https://codesandbox.io/embed/55wp11lv4)

Contextual isn't limited to reading context and store patterns, it also helps you to create and share providers.

* [moduleContext](https://github.com/drcmda/react-contextual/blob/master/API.md#modulecontext) creates a global provider and injects it into a component
* [namedContext](https://github.com/drcmda/react-contextual/blob/master/API.md#namedcontext) creates a unique provider bound to a components lifecycle
* [transformContext](https://github.com/drcmda/react-contextual/blob/master/API.md#transformcontext) transforms existing providers (like a declarative middleware)
* [helper functions](https://github.com/drcmda/react-contextual/blob/master/API.md#imperative-context-handling) allow you to control a context by yourself

#### Custom providers & transforms

```jsx
import { subscribe, moduleContext, transformContext } from 'react-contextual'

const Theme = moduleContext()(
    ({ context, color, children }) => <context.Provider value={{ color }} children={children} />
)

const Invert = transformContext(Theme)(
    ({ context, color, children }) => <context.Provider value={invert(color)} children={children} />
)

const Write = subscribe(Theme)(
    ({ color, text }) => <span style={{ color }}>{text}</span>
)

const App = () => (
    <Theme color="red">
        <Write text="hello" />
        <Invert>
            <Write text="world" />
        </Invert>
    </Theme>,
)
```

#### With decorator

```jsx
@moduleContext()
class Theme extends React.PureComponent {
    // ...
}

@transformContext(Theme)
class Invert extends React.PureComponent {
    // ...
}

@subscribe(Theme)
class Say extends React.PureComponent {
    // ...
}
```

---

[API](https://github.com/drcmda/react-contextual/blob/master/API.md) | [Changelog](https://github.com/drcmda/react-contextual/blob/master/CHANGELOG.md) | [Pitfalls using context raw](https://github.com/drcmda/react-contextual/blob/master/PITFALLS.md)

## Who is using it

[![AWV](/assets/corp-awv.png)](https://github.com/awv-informatik)
