export const filename = __filename
export const demoboardHelpers = {
  'App.js': require('!raw-loader!./minimal-example/App.js'),
  'index.js': require('!raw-loader!./minimal-example/index.js'),
  'pages.js': require('!raw-loader!./minimal-example/pages.js'),
  'pages/Reference.js': require('!raw-loader!./minimal-example/Reference.js'),
}

The Minimal Example
===================

After spinning up a fresh app with [create-react-app](https://github.com/facebook/create-react-app), start by installing the `navi` and `react-navi` packages:

```bash
npm install --save navi react-navi
```

This leaves you with just three short steps to creating an app with asynchronous content and smooth transitions between pages.


Step 1: Declare some pages
--------------------------

To declare your pages, you'll use Navi's `createSwitch()` and `createPage()` functions. Switches are used to map URL paths to pages. Pages represent individual locations that you can navigate to.

```js
//--- pages.js <-- pages.js
//--- pages/Reference.js <-- pages/Reference.js
//--- index.js <-- index.js
//--- App.js <-- App.js
```

As you'll see later, your content can be *anything*. You can return markdown, JSON, or even arbitrary functions! But `react-navi` has special support for React elements and components, so let's start by defining the content that way.

But what about the `/reference` page? It's not returning an element or component. It's returning a *Promise* to a component -- and this is where Navi shines. When the user clicks the "API reference" link, instead of immediately rendering a blank page, Navi will wait until `reference.js` has loaded --  and *then* it'll render the page.

```js
//---
  editorFilename: "/pages/Reference.js"
//--- pages.js <-- pages.js
//--- pages/Reference.js <-- pages/Reference.js
//--- index.js <-- index.js
//--- App.js <-- App.js
```

Step 2: Create `navigation`
---------------------------

Navi does all of the hard work within a `Navigation` object. This is where Navi watches for history events, matches URLs to pages and content, and turns all of this info into an object that you can use.

To create a `Navigation`, just call `createBrowserNavigation()` within `index.js`, passing in the `pages` object that you defined earlier. Once you have a `Navigation`, just wait for its initial content to become available -- and then render it!

```js
//---
  editorFilename: "/index.js"
//--- pages.js <-- pages.js
//--- pages/Reference.js <-- pages/Reference.js
//--- index.js <-- index.js
//--- App.js <-- App.js
```


Step 3: Render your route
-------------------------

The `navigation` object that you just passed to `<App>` contains all of the information that you need to render your app. And while you *could* consume all of that information yourself, it's far simpler to just use Navi's built in components.

To start out, you'll only need three components:

- `<NavProvider>`, which wraps around your entire application
- `<NavRoute>`, which renders the current page's content
- `<NavNotFoundBoundary>`, which renders a 404 message when it catches a `NotFoundError` (as thrown by `<NavRoute>`)

Here's an example:

```js
//---
  editorFilename: "/App.js"
//--- pages.js <-- pages.js
//--- pages/Reference.js <-- pages/Reference.js
//--- index.js <-- index.js
//--- App.js <-- App.js
```

And that's it -- you've built a working app with asynchronous content! Of course, this tiny app is just an example, but Navi handles real-world apps with ease. In fact, [Frontend Armory](https://frontarm.com) is built with Navi.

Now that you have a working app, let's add a one more tweak as a bonus step...


Bonus: Loading indicators
-------------------------

Navi doesn't render the new pages until they have loaded, so there can sometimes be a large delay between clicking a link seeing the result. In cases like this, it's important to keep the user in the loop. And to do so, you can wrap your route with a `<NavLoading>` component:

```js
//---
  editorFilename: "/App.js"
//--- pages.js
/**
 * This version of pages.js adds delays to content loading to
 * give the loading bar time to display.
 */

import { createPage, createSwitch } from 'navi'
import * as React from 'react'
import { NavLink } from 'react-navi'

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

export default createSwitch({
  paths: {
    '/': createPage({
      title: "Navi",
      getContent: async () => {
        await delay(1000)

        return (
          <div>
            <h2>Navi</h2>
            <p>A router/loader for React</p>
            <nav><NavLink href='/reference'>API Reference</NavLink></nav>
          </div>
        )
      }
    }),

    '/reference': createPage({
      title: "API Reference",
      getContent: async () => {
        await delay(1000)
        return import('./pages/Reference')
      }
    }),
  }
})
//--- pages/Reference.js <-- pages/Reference.js
//--- index.js <-- index.js
//--- App.js
import * as React from 'react'
import { NavLink, NavProvider, NavRoute, NavLoading, NavNotFoundBoundary } from 'react-navi'
import BusyIndicator from 'react-busy-indicator'

class App extends React.Component {
  render() {
    return (
      <NavProvider navigation={this.props.navigation}>
        <NavLoading>
          {isLoading =>
            <div className="App">
              <BusyIndicator
                color="#ff0000"
                delayMs={333}
                isBusy={isLoading}
              />
              <header className="App-header">
                <h1 className="App-title">
                  <NavLink href='/'>Navi</NavLink>
                </h1>
              </header>
              <NavNotFoundBoundary render={renderNotFound}>
                <NavRoute />
              </NavNotFoundBoundary>
            </div>
          }
        </NavLoading>
      </NavProvider>
    );
  }
}

function renderNotFound() {
  return (
    <div className='App-error'>
      <h1>404 - Not Found</h1>
    </div>
  )
} 

export default App;
```

The `<NavLoading>` component accepts a render function as its children, to which it passes a boolean that indicates whether its nested `<NavRoute>` component is waiting for  content. You're free to render this boolean however you'd like, but you can save yourself the trouble by importing and using `react-busy-indicator` -- the same loading bar used on this site.


What next?
----------

If you'd like your site's content to be indexed by search engines and shared on social media, take a look at the [Static Rendering](../static-rendering) guide -- it'll only take you 2 minutes to complete!

<!--But Navi isn't just for websites -- it works great for Single Page Apps too! So if you're building an SPA, head on over to the [Authenticated Routes](../authenticated-routes) guide and learn how to hide protected content from unauthenticated users.-->