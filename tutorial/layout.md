# Layout

In this section we will explore the following:
* [Layout elements](#layout-elements)
* [Main layout operations](#main-layout-operations)
* [Main content and routing](#main-content-and-routing)

## Layout elements

Outer layouts define a wrapper around the content being rendered. This is very convenient when multiple pages need to share a common layout or for defining global application layouts.

To assign an outer layout to a widget, we specify the `outerLayout` attribute and pass in the predefined application layout.

A layout is a simple widget tree with `ContentPlaceholder` elements that are used to specify content insertion points.

## App layout

Our app is using the default layout as defined in our project scaffold. The layout consists of a collapsable sidebar, a header and the main area.

Let's examine the default layout:

#### app/layout/index.js

```
import {HtmlElement, Link, Button} from 'cx/widgets';
import {ContentPlaceholder} from 'cx/ui';
import Controller from "./Controller";

export default <cx>
    <div
        controller={Controller}
        class={{
            "layout": true,
            "nav": {bind: "layout.aside.open"}
        }}
    >
        <main class="main" onMouseDownCapture="onMainClick">
            <ContentPlaceholder />
        </main>
        <header class="header">
            <i
                class={{
                    hamburger: true,
                    open: {bind: 'layout.aside.open'}
                }}
                onClick={(e, {store}) => {
                    store.toggle('layout.aside.open');
                }}
            />
            <ContentPlaceholder name="header"/>
        </header>
        <aside class="aside">
            <h1>Cx App</h1>
            <dl>
                <dt>
                    App
                </dt>
                <dd>
                    <Link href="~/" url:bind="url">
                        Home
                    </Link>
                </dd>
                <dd>
                    <Link href="~/dashboard" url:bind="url">
                        Dashboard
                    </Link>
                </dd>
                <dd>
                <Link href="~/about" url:bind="url">
                    About
                </Link>
                </dd>
            </dl>
            <dl>
                <dt>
                    Admin
                </dt>
                <dd>
                    <Link href="~/users" url:bind="url" match="prefix">
                        Users
                    </Link>
                </dd>
            </dl>
        </aside>
    </div>
</cx>
```

The layout consists of three main sections: `main`, `header` and `aside` (for the sidebar).
It is interesting to notice that our `header` and `aside` elements come after the `main` element. The reason is that our scaffold app is using `fixed` positioning for `header` and `aside` elements, and this ensures the correct stacking order: `main` at the bottom (gray background), `aside` at the top (white sidebar) and the `header` in between (over `main` and under `aside`).

<a href="https://github.com/codaxy/cxjs-home-expenses-app-tutorial/blob/master/tutorial/screenshots/default-layout.PNG">
    <img src="https://github.com/codaxy/cxjs-home-expenses-app-tutorial/blob/master/tutorial/screenshots/default-layout.PNG" alt="Default layout" />
</a>

Default layout




## Links

We use the [`Link`](https://docs.cxjs.io/widgets/links) Widget to define our links. `href` attribute represents the Url to the link's target location. Since we are using relative paths, the Url begins with the `~/` prefix which will in our current local setup automatically get replaced with the app's root url: `http://localhost:8088/`.

The following code-changes to the sidebar will define the links for our app and set the new sidebar header text to "Home Expenses":

```
    ...
    import {ContentPlaceholder, bind} from 'cx/ui';
    ...
        <aside class="aside">
            <h1>Home Expenses</h1>
            <dl>
                <dt>
                    <Link href="~/dashboard" url={bind("url")}>
                        Dashboard
                    </Link>
                </dt>
                <dt>
                    <Link href="~/entry/new" url={bind("url")}>
                        Add Expense
                    </Link>
                </dt>
                <dt>
                    <Link href="~/log" url={bind("url")}>
                        Log
                    </Link>
                </dt>
            </dl>
        </aside>
    ...
```

We import the `bind` function from `cx/ui`, so we can use functional binding sintax as described in [Core concepts](https://github.com/codaxy/cxjs-home-expenses-app-tutorial/blob/master/tutorial/core-concepts.md#two-way-data-binding-bind).

We set the `url` attribute as a binding to the `'url'` value available in the Store. If `href` matches `url`, additional CSS class `active` is applied to indicate the link that is currently active. If you are wondering where did this `'url'` value inside the Store come from, the answer lies in this piece of code:

#### app/index.js
```
...
//routing
Url.setBaseFromScript('app.js');
History.connect(store, 'url');
...
```
The `History` module is used for working with HTML5 `pushState` navigation. It basically keeps track of the user's navigation history, which enables the use of browser's `back` and `forward` commands.

`History.connect(store, 'url');` makes sure that, as the user navigates the site, the current Url is always kept up to date with the Store value called `'url'`. Mistery solved!




Next, let's change the default header color:

#### app/layout/index.scss

```
$header-color: #0d8aee;
```

Cx uses Sass preprocessor for styling. For more info on styling Cx apps, checkout the [docs page](https://docs.cxjs.io/concepts/css).

Our app should now like like this:

<a href="https://github.com/codaxy/cxjs-home-expenses-app-tutorial/blob/master/tutorial/screenshots/add-links.PNG">
    <img src="https://github.com/codaxy/cxjs-home-expenses-app-tutorial/blob/master/tutorial/screenshots/add-links.PNG" alt="Added links" />
</a>







Now let's examine one of the predefined routes, to see how we actually place dynamic content into `ContentPlaceholders` in our `header` and `main` sections:

#### app/routes/dashboard/index.js
```
...

export default <cx>
    <h2 putInto="header">Dashboard</h2>

    <FlexCol spacing="large" controller={Controller}>
        ...
    </FlexCol>
</cx>
```

If we examine the Cx element that is exported in this file, we can see there is a `h2` header with `putInto` attribute, and another `FlexCol` element that holds the rest of the content.

The value of the `putInto` attribute actually represents the `name` of the `ContentPlaceholder`, or the insertion point for our `h2` elements. This means, the `h2` elements will get inserted into the `<ContentPlaceholder name='header' />` element, which is inside the `header` element in our layout.

The `FlexCol` element, that has no `putInto` attribute, will be inserted into the `ContentPlaceholder` inside the `main` element in our layout. This is because the nameless `ContentPlaceholder` is used as the default insertion point for all of the content without the `putInto` attribute.


## Main application Routes

The `app/routes/index.js` file is where the actual routes are defined. 

The `Route` widget is a pure container element which renders only if current url matches the assigned route.

Let's see how to use them:

#### app/routes/index.js
```
import { Route, RedirectRoute, PureContainer, Section, Sandbox } from 'cx/widgets';
import { FirstVisibleChildLayout, bind, expr } from 'cx/ui'

import AppLayout from '../layout';

import Dashboard from './dashboard';
import Entry from './entry';
import Log from './log';


export default <cx>
    <PureContainer layout={FirstVisibleChildLayout}>
        <Sandbox
            key={bind("url")}
            storage={bind("pages")}
            outerLayout={AppLayout}
            layout={FirstVisibleChildLayout}
        >
            <RedirectRoute
                route="~/"
                url={bind("url")}
                redirect="~/dashboard"
            />

            <Route route="~/entry/:id" url={bind("url")}>
                <Entry />
            </Route>

            <Route route="~/log" url={bind("url")}>
                <Log />
            </Route>

            <Route route="~/dashboard" url={bind("url")}>
                <Dashboard />
            </Route>

            <Section title="Page Not Found" mod="card">
                This page doesn't exists. Please check your URL.
            </Section>

        </Sandbox>
    </PureContainer>
</cx>
```






## Main content and routing