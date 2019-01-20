# Code Splitting

We use `react-loadable` package to provide an easier HOC api and import async components into routes or pages/components.

## How to import a component inside another (component/page)

We simply add loadable and then use a Loading component as fallback while the real one loads:

```
import loadable from 'react-loadable'

// loading view
const LoadingComponent = () => <h3>please wait...</h3>

// async component
const AsyncComponent = loadable({
  loader: () =>
    import(/* webpackChunkName: "async-wrapper" */ './AnotherAsync'),
  loading: LoadingComponent
})
```

Then we use the AsyncComponent as our HOC component, which will switch from Loading to loaded once ready (including an async chunk).

This improves loading performance and is very useful for outside-the-fold elements.

Example:
```
import React from 'react'

export default class Home extends Component {
  render () {
    return (
      <React.Fragment>
        <div styleName='home'>
          <If condition={true}>A conditional statement result can go here</If>
          <p>Hello world!</p>
        </div>
        <AsyncComponent /> <!-- we load the component here -->
      </React.Fragment>
    )
  }
}
```

## Async Routes

To do the same with routes, we create HOC wrappers in the `routes.js` file:

```
import Home from './pages/Home'


import loadable from 'react-loadable'

// loading view
const LoadingComponent = () => <h3>please wait...</h3>

// async component
const AsyncWrapper = loadable({
  loader: () =>
    import(/* webpackChunkName: "async-page" */ './components/AsyncComponent/AsyncComponent'),
  loading: LoadingComponent
})

export default [
  {
    path: '/new-checkout',
    component: Home,
    routes: [
      {
        exact: true,
        component: Home
      }
    ]
  },
  {
    path: '/async',
    component: AsyncWrapper,
    routes: [
      {
        exact: true,
        component: AsyncWrapper
      }
    ]
  }
]
```

The async route will be loaded async only when visited. (prefetching is possible).

It will also load a chunk name called `AsyncComponent.[hash].chunk.js` which is controlled via the webpack.config.js file using the output:  
`chunkFilename: '[name].[hash].chunk.js',`

This allows for easier understanding when developing and can be turned off if needed in production.
