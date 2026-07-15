# Queries

- [Queries](#queries)
  - [`useQuery`](#usequery)
  - [Query keys](#query-keys)
  - [Query functions](#query-functions)
  - [The query result returned by `useQuery`](#the-query-result-returned-by-usequery)
  - [Using `isFetching`](#using-isfetching)
  - [`useIsFetching` hook](#useisfetching-hook)

---

---

## `useQuery`

The `useQuery` hook handles the fetching of data. It is called whenever you need to fetch data. It accepts

- a **unique key** for the query and
- **A function that returns a promise** that:
  - Resolves the data, or
  - Throws an error.

---

---

## Query keys

The unique key is used to refetch, cache, and share your query internally.

**Note:** Query Keys (and Mutation Keys) need to be an Array.

In the previous version of React Query we could do this:

```js
useQuery("key", promiseBasedFunc);
```

Starting with V4, which is called Tanstack Query, the query key must be an array:

```js
useQuery(["key"], promiseBasedFunc);
```

---

---

## Query functions

A query function can be literally any function that returns a promise. The promise that is returned should either resolve the data or throw an error.

All of the following are valid query function configurations:

```js
useQuery({ queryKey: ["todos"], queryFn: fetchAllTodos });
```

```js
useQuery({ queryKey: ["todos", todoId], queryFn: () => fetchTodoById(todoId) });
```

```js
useQuery({
  queryKey: ["todos", todoId],
  queryFn: async () => {
    const data = await fetchTodoById(todoId);
    return data;
  },
});
```

```js
useQuery({
  queryKey: ["todos", todoId],
  queryFn: ({ queryKey }) => fetchTodoById(queryKey[1]),
});
```

---

---

## The query result returned by `useQuery`

```js
const result = useQuery({
  queryKey: ["todos"],
  queryFn: fetchTodoList,
});
```

The query result object contains a few very important states you'll need to be aware of to be productive. A query can only be in one of the following states at any given moment:

- `isLoading` or `status === 'loading'` - The query has no data yet
- `isError` or `status === 'error'` - The query encountered an error
- `isSuccess` or `status === 'success'` - The query was successful and data is available

Beyond those primary states, more information is available depending on the state of the query:

- `error` - If the query is in an `isError` state, the error is available via the `error` property.
- `data` - If the query is in an `isSuccess` state, the data is available via the `data` property.
- `isFetching` - In any state, if the query is fetching at any time (including background refetching) `isFetching` will be `true`.

For most queries, it's usually sufficient to check for the `isLoading` state, then the `isError` state, then finally, assume that the `data` is available and render the successful state:

Here is an example:

```js
// api/api.js
export const fetchData = async () => {
  const response = await fetch("http://localhost:8000/result");

  if (!response.ok)
    return new Error(`Can't fetch data. Status: ${response.status}`);

  const data = await response.json();

  return data;
};
```

```js
// App.jsx
import { fetchData } from "./api/api";
import { useQuery } from "@tanstack/react-query";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  if (query.isLoading) return <h1>Loading...</h1>;
  if (query.isError)
    return <h1>Error in fetching data: {query.error.message}</h1>;

  return (
    <div>
      {query?.data?.map((item) => {
        return (
          <h3 key={item.id}>
            {item.first_name} {item.last_name}
          </h3>
        );
      })}
    </div>
  );
};

export default App;
```

Here is the same example, this time using the `status` property returned by `useQuery`, instead of using booleans:

```js
// App.jsx
import { fetchData } from "./api/api";
import { useQuery } from "@tanstack/react-query";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  return (
    <div>
      {query?.data?.map((item) => {
        return (
          <h3 key={item.id}>
            {item.first_name} {item.last_name}
          </h3>
        );
      })}
    </div>
  );
};

export default App;
```

---

---

## Using `isFetching`

A query's `status === 'loading'` state is sufficient enough to show the initial hard-loading state for a query, but sometimes you may want to display an additional indicator that a query is refetching in the background. Also, queries can be in `state: 'loading'`, but `fetchStatus: 'paused'` if they are mounting for the first time, and you have no network connection.

Queries supply you with an `isFetching` boolean that you can use to show that it's in a fetching state, regardless of the state of the `status` variable.

Here is an example of using `isFetching`. Note that we used `setTimeout` in the `api.js` file to simulate a longer response time:

```js
// api/api.js
export const fetchData = () => {
  return new Promise((resolve) =>
    setTimeout(async () => {
      const response = await fetch("http://localhost:8000/result");

      if (!response.ok)
        return new Error(`Can't fetch data. Status: ${response.status}`);

      const data = await response.json();

      resolve(data);
    }, 1000),
  );
};
```

```js
// App.jsx
import { fetchData } from "./api/api";
import { useQuery } from "@tanstack/react-query";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });
  console.log(query.status);

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  return (
    <div>
      {query.isFetching ? <div>Refreshing...</div> : null}
      {query?.data?.map((item) => {
        return (
          <h3 key={item.id}>
            {item.first_name} {item.last_name}
          </h3>
        );
      })}
    </div>
  );
};

export default App;
```

---

---

## `useIsFetching` hook

In addition to individual query loading states, if you would like to show a global loading indicator when any queries are fetching (including in the background), you can use the `useIsFetching` hook.

Here, we create a component called `GlobalRefetch`. Even though we don't use the `useQuery` hook in this component, it still can show when data is fetched using the `useIsFetching` hook:

```js
// components/GlobalRefetch.jsx
import { useIsFetching } from "@tanstack/react-query";

const GlobalRefetch = () => {
  const isFetching = useIsFetching();

  return isFetching ? (
    <div>Queries are fetching in the background...</div>
  ) : null;
};

export default GlobalRefetch;
```

```js
// App.jsx
import { fetchData } from "./api/api";
import { useQuery } from "@tanstack/react-query";
import GlobalRefetch from "./components/GlobalRefetch";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });
  console.log(query.status);

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  return (
    <div>
      <GlobalRefetch />

      {query?.data?.map((item) => {
        return (
          <h3 key={item.id}>
            {item.first_name} {item.last_name}
          </h3>
        );
      })}
    </div>
  );
};

export default App;
```

---

---
