# Server Rendering

Server rendering is a bit different than in a client because you'll want
to:

- Send `500` responses for errors
- Send `30x` responses for redirects
- Fetch data before rendering (and use the router to help you do it)

To facilitate these needs, you drop one level lower than the [`<Router/>`](/docs/api/Router.md)
API with:  

- `createLocation` from the history package
- `match` to match the routes to a location without rendering
- `RoutingContext` for synchronous rendering of route components

It looks something like this with an imaginary JavaScript server:

```js
import createLocation from 'history/lib/createLocation'
import { RoutingContext, match } from 'react-router'
import routes from './routes'
import { renderToString } from 'react-dom/server'

serve((req, res) => {
  let location = createLocation(req.url)

  match({ routes, location }, (error, redirectLocation, renderProps) => {
    if (redirectLocation)
      res.redirect(301, redirectLocation.pathname + redirectLocation.search)
    else if (error)
      res.send(500, error.message)
    else if (renderProps == null)
      res.send(404, 'Not found')
    else
      res.send(renderToString(<RoutingContext {...renderProps}/>))
  })
})
```

For data loading, you can use the `renderProps` argument to build whatever
convention you want--like adding static `load` methods to your route
components, or putting data loading functions on the routes, it's up to
you.