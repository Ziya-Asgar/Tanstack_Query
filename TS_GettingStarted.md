# Getting Started

**React Query** is also known as TanStack Query. TanStack query is an asynchronous state management library.

To install TanStack's React query, we use the below command:

```
npm i @tanstack/react-query
```

We can also install `react-query-devtools`, which could simplify our work with React Query. It is difficult to debug data fetching, that is why you need to install `react-query-devtools`. It will help you to see and understand how React Query does data fetching.:

```
npm i @tanstack/react-query-devtools
```

To set up the Tanstack query, we use the below steps:

- import `QueryClient`, and `QueryClientProvider` from `"@tanstack/react-query"` library
- Create a `new QueryClient()`
- Put your `<App />` component inside opening and closing `QueryClientProvider` tags
- `QueryClientProvider` has a `client` property. Make that `client` property equal the newly created `QueryClient()`

```js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <ReactQueryDevtools />
    </QueryClientProvider>
  </React.StrictMode>,
);
```

`QueryClient` keeps the cache of all queries made and tracks their state, while `QueryClientProvider` makes `QueryClient` available anywhere in your application.

---

---
