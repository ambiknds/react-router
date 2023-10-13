## Running React Router
https://reactrouter.com/en/main/start/tutorial

https://tailwindcss.com/docs/guides/vite

https://github.com/hiteshchoudhary/react-english/tree/main/07reactRouter

##create github api json data
https://api.github.com/users/ambiknds
----------------------------------------
##createbrowserrouter
This is the recommended router for all React Router web projects. It uses the DOM History API to update the URL and manage the history stack.

It also enables the v6.4 data APIs like loaders, actions, fetchers and more.

Due to the decoupling of fetching and rendering in the design of the data APIs, you should create your router outside of the React tree with a statically defined set of routes. For more information on this design, please see the Remixing React Router blog post and the When to Fetch conference talk.

import * as React from "react";
import * as ReactDOM from "react-dom";
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

import Root, { rootLoader } from "./routes/root";
import Team, { teamLoader } from "./routes/team";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    children: [
      {
        path: "team",
        element: <Team />,
        loader: teamLoader,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
------------------------------------
##createRoutesFromElements
createRoutesFromElements is a helper that creates route objects from <Route> elements. It's useful if you prefer to create your routes as JSX instead of objects.

import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

// You can do this:
const router = createBrowserRouter(
  createRoutesFromElements(
    <Route path="/" element={<Root />}>
      <Route path="dashboard" element={<Dashboard />} />
      <Route path="about" element={<About />} />
    </Route>
  )
);

// Instead of this:
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        path: "dashboard",
        element: <Dashboard />,
      },
      {
        path: "about",
        element: <About />,
      },
    ],
  },
]);
It's also used internally by <Routes> to generate a route objects from its <Route> children.
---------------------------------------------
##Route
Routes are perhaps the most important part of a React Router app. They couple URL segments to components, data loading and data mutations. Through route nesting, complex application layouts and data dependencies become simple and declarative.

Routes are objects passed to the router creation functions:

const router = createBrowserRouter([
  {
    // it renders this element
    element: <Team />,

    // when the URL matches this segment
    path: "teams/:teamId",

    // with this data loaded before rendering
    loader: async ({ request, params }) => {
      return fetch(
        `/fake/api/teams/${params.teamId}.json`,
        { signal: request.signal }
      );
    },

    // performing this mutation when data is submitted to it
    action: async ({ request }) => {
      return updateFakeTeam(await request.formData());
    },

    // and renders this element in case something went wrong
    errorElement: <ErrorBoundary />,
  },
]);
You can also declare your routes with JSX and createRoutesFromElements, the props to the element are identical to the properties of the route objects:

const router = createBrowserRouter(
  createRoutesFromElements(
    <Route
      element={<Team />}
      path="teams/:teamId"
      loader={async ({ params }) => {
        return fetch(
          `/fake/api/teams/${params.teamId}.json`
        );
      }}
      action={async ({ request }) => {
        return updateFakeTeam(await request.formData());
      }}
      errorElement={<ErrorBoundary />}
    />
  )
);
Neither style is discouraged and behavior is identical. For the majority of this doc we will use the JSX style because that's what most people are accustomed to in the context of React Router.

When using RouterProvider, if you do not wish to specify a React element (i.e., element={<MyComponent />}) you may specify a Component instead (i.e., Component={MyComponent}) and React Router will call createElement for you internally. You should only do this for RouterProvider applications though since using Component inside of <Routes> will de-optimize React's ability to reuse the created element across renders.
-----------------------------------------
#Link
A <Link> is an element that lets the user navigate to another page by clicking or tapping on it. In react-router-dom, a <Link> renders an accessible <a> element with a real href that points to the resource it's linking to. This means that things like right-clicking a <Link> work as you'd expect. You can use <Link reloadDocument> to skip client side routing and let the browser handle the transition normally (as if it were an <a href>).

import * as React from "react";
import { Link } from "react-router-dom";

function UsersIndexPage({ users }) {
  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            <Link to={user.id}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
A relative <Link to> value (that does not begin with /) resolves relative to the parent route, which means that it builds upon the URL path that was matched by the route that rendered that <Link>. It may contain .. to link to routes further up the hierarchy. In these cases, .. works exactly like the command-line cd function; each .. removes one segment of the parent path.
----------------------------------------------
#NavLink
A <NavLink> is a special kind of <Link> that knows whether or not it is "active" or "pending". This is useful when building a navigation menu, such as a breadcrumb or a set of tabs where you'd like to show which of them is currently selected. It also provides useful context for assistive technology like screen readers.

import { NavLink } from "react-router-dom";

<NavLink
  to="/messages"
  className={({ isActive, isPending }) =>
    isPending ? "pending" : isActive ? "active" : ""
  }
>
  Messages
</NavLink>;
---------------------------------
#useParams

The useParams hook returns an object of key/value pairs of the dynamic params from the current URL that were matched by the <Route path>. Child routes inherit all params from their parent routes.

import * as React from 'react';
import { Routes, Route, useParams } from 'react-router-dom';

function ProfilePage() {
  // Get the userId param from the URL.
  let { userId } = useParams();
  // ...
}

function App() {
  return (
    <Routes>
      <Route path="users">
        <Route path=":userId" element={<ProfilePage />} />
        <Route path="me" element={...} />
      </Route>
    </Routes>
  );
}



-----------------------------------------------------
##useLoaderData
This hook provides the value returned from your route loader.

import {
  createBrowserRouter,
  RouterProvider,
  useLoaderData,
} from "react-router-dom";

function loader() {
  return fetchFakeAlbums();
}

export function Albums() {
  const albums = useLoaderData();
  // ...
}

const router = createBrowserRouter([
  {
    path: "/",
    loader: loader,
    element: <Albums />,
  },
]);

ReactDOM.createRoot(el).render(
  <RouterProvider router={router} />
);
After route actions are called, the data will be revalidated automatically and return the latest result from your loader.

Note that useLoaderData does not initiate a fetch. It simply reads the result of a fetch React Router manages internally, so you don't need to worry about it refetching when it re-renders for reasons outside of routing.

This also means data returned is stable between renders, so you can safely pass it to dependency arrays in React hooks like useEffect. It only changes when the loader is called again after actions or certain navigations. In these cases the identity will change (even if the values don't).

You can use this hook in any component or any custom hook, not just the Route element. It will return the data from the nearest route on context.

