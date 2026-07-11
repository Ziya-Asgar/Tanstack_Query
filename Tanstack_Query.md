# Tanstack Query (formerly React Query) notes

<hr>

- [Tanstack Query (formerly React Query) notes](#tanstack-query-formerly-react-query-notes)
  - [Some useful links](#some-useful-links)
  - [How to install and set up Tanstack Query](#how-to-install-and-set-up-tanstack-query)
- [Query and Mutation](#query-and-mutation)
  - [Query](#query)
    - [Query keys](#query-keys)
    - [Query functions](#query-functions)
    - [The query result returned by `useQuery`](#the-query-result-returned-by-usequery)
    - [Using `isFetching`](#using-isfetching)
    - [`useIsFetching` hook](#useisfetching-hook)
  - [Mutation](#mutation)
    - [`useMutation` callback functions](#usemutation-callback-functions)
    - [Updating the query after the mutation](#updating-the-query-after-the-mutation)
      - [Using `setQueryData`](#using-setquerydata)
      - [Using `invalidateQueries`](#using-invalidatequeries)
    - [The `mutateAsync` method](#the-mutateasync-method)
  - [Query results in pages](#query-results-in-pages)
  - [Query with infinite scrolling](#query-with-infinite-scrolling)

<hr>

## Some useful links

- https://www.youtube.com/watch?v=k1tus-TmqCE
- https://blog.logrocket.com/deep-dive-mutations-tanstack-query/

## How to install and set up Tanstack Query

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

# Query and Mutation

2 main things that you can do with React query is a **query** and a **mutation**.

- A **query** is getting data from somewhere. For example, fetching data from the server is an act of performing a query.
- A **mutation** is changing some data. For example, creating a new post.

<hr>

## Query

The `useQuery` hook handles the fetching of data. It is called whenever you need to fetch data. It accepts

- a **unique key** for the query and
- **A function that returns a promise** that:
  - Resolves the data, or
  - Throws an error.

### Query keys

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

### Query functions

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

### The query result returned by `useQuery`

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

<hr>

### Using `isFetching`

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

### `useIsFetching` hook

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

## Mutation

To have a mutation in Tanstack Query, we use the `useMutation` hook. This hook accepts a mutation function (`mutationFn`) and returns several useful properties, including the `mutate` method. We call the `mutate` method to use the mutation function to create, update or delete data on the server.

Here is an example, where we first create and export a function that sends a POST request. Then, we initialize the `useMutation` hook and send the data to the server with the `mutate` method.

```js
// api/api.js
export const addData = async (post) => {
  const response = await fetch("http://localhost:8000/result", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(post),
  });

  return response.json();
};
```

```js
// App.jsx
import { fetchData, addData } from "./api/api";
import { useMutation, useQuery } from "@tanstack/react-query";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  const mutation = useMutation({
    mutationFn: addData,
  });

  const toBePosted = {
    id: query?.data?.length + 1,
    first_name: "Name",
    last_name: "Surname",
    email: "name-surname@example.com",
  };

  const handleSend = () => {
    mutation.mutate(toBePosted);
  };

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  if (mutation.isLoading) return <h1>Sending data...</h1>;
  if (mutation.isError)
    return <h1>Error in sending data: {mutation.error.message}</h1>;

  return (
    <div>
      <button onClick={handleSend}>Send Data</button>
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

### `useMutation` callback functions

If we want to directly update the data at any point in the mutation lifecycle, `useMutation` provides us with callback functions for side effects:

- `onMutate`: Fires before the mutation function fires
- `onError`: Will fire if the mutation fails
- `onSuccess`: Fires when the mutation is successful
- `onSettled`: Will fire when the mutation succeeds or fails

```js
// App.jsx
import { fetchData, addData } from "./api/api";
import { useMutation, useQuery } from "@tanstack/react-query";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  const mutation = useMutation({
    mutationFn: addData,
    onMutate: () => {
      console.log("mutation is starting");
    },
    onSuccess: () => {
      console.log("mutation was successful");
    },
    onError: () => {
      console.log("Error while mutating");
    },
    onSettled: () => {
      console.log("mutation ended");
    },
  });

  const toBePosted = {
    id: query?.data?.length + 1,
    first_name: "Name",
    last_name: "Surname",
    email: "name-surname@example.com",
  };

  const handleSend = () => {
    mutation.mutate(toBePosted);
  };

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  if (mutation.isLoading) return <h1>Sending data...</h1>;
  if (mutation.isError)
    return <h1>Error in sending data: {mutation.error.message}</h1>;

  return (
    <div>
      <button onClick={handleSend}>Send Data</button>
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

### Updating the query after the mutation

#### Using `setQueryData`

At this point, our cache (the result of our query) doesn't get updated after our mutation. We have to refresh our app to get the new - mutated - data. How can we get the data after mutation without refreshing? One way is to use the `setQueryData` to directly update our cache after the successful mutation. To use the `setQueryData`, we need to export the `queryClient` from the main (main.jsx or index.js) component and import it in the component where we are going to use the `setQueryData`. `queryClient` gives us access to interact with the cache.

This is the syntax for `setQueryData`: `setQueryData(<queryKey>, <updater>)`. Updater can be either new data as a value or a function that returns new data.

Here is an example:

```js
// main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

export const queryClient = new QueryClient();

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <ReactQueryDevtools />
    </QueryClientProvider>
  </React.StrictMode>,
);
```

```js
// App.jsx
import { fetchData, addData } from "./api/api";
import { useMutation, useQuery } from "@tanstack/react-query";
import { queryClient } from "./main";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  const mutation = useMutation({
    mutationFn: addData,
    onMutate: () => {
      console.log("mutation is starting");
    },
    onSuccess: () => {
      console.log("mutation was successful");
      queryClient.setQueryData(["users"], [...query.data, toBePosted]);
    },
    onError: () => {
      console.log("Error while mutating");
    },
    onSettled: () => {
      console.log("mutation ended");
    },
  });

  const toBePosted = {
    id: query?.data?.length + 1,
    first_name: "Name",
    last_name: "Surname",
    email: "name-surname@example.com",
  };

  const handleSend = () => {
    mutation.mutate(toBePosted);
  };

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  if (mutation.isLoading) return <h1>Sending data...</h1>;
  if (mutation.isError)
    return <h1>Error in sending data: {mutation.error.message}</h1>;

  return (
    <div>
      <button onClick={handleSend}>Send Data</button>
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

#### Using `invalidateQueries`

Another way of updating the cache is to invalidate the query, so that fetching happens again and we get new data with our query. To invalidate the cache, we use the `invalidateQueries` method from the `queryClient`. This way, we automatically get the new data after the mutation:

```js
// App.jsx
import { fetchData, addData } from "./api/api";
import { useMutation, useQuery } from "@tanstack/react-query";
import { queryClient } from "./main";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  const mutation = useMutation({
    mutationFn: addData,
    onMutate: () => {
      console.log("mutation is starting");
    },
    onSuccess: () => {
      console.log("mutation was successful");
      queryClient.invalidateQueries(["users"]);
    },
    onError: () => {
      console.log("Error while mutating");
    },
    onSettled: () => {
      console.log("mutation ended");
    },
  });

  const toBePosted = {
    id: query?.data?.length + 1,
    first_name: "Name",
    last_name: "Surname",
    email: "name-surname@example.com",
  };

  const handleSend = () => {
    mutation.mutate(toBePosted);
  };

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  if (mutation.isLoading) return <h1>Sending data...</h1>;
  if (mutation.isError)
    return <h1>Error in sending data: {mutation.error.message}</h1>;

  return (
    <div>
      <button onClick={handleSend}>Send Data</button>
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

### The `mutateAsync` method

The `mutateAsync` method lets us manipulate data asynchronously. `mutate` doesn't return anything, while `mutateAsync` returns a Promise containing the result of the mutation.

```js
import { fetchData, addData } from "./api/api";
import { useMutation, useQuery } from "@tanstack/react-query";
import { queryClient } from "./main";

const App = () => {
  const query = useQuery({
    queryKey: ["users"],
    queryFn: fetchData,
  });

  const mutation = useMutation({
    mutationFn: addData,
    onMutate: () => {
      console.log("mutation is starting");
    },
    onSuccess: () => {
      console.log("mutation was successful");
      queryClient.invalidateQueries(["users"]);
    },
    onError: () => {
      console.log("Error while mutating");
    },
    onSettled: () => {
      console.log("mutation ended");
    },
  });

  const toBePosted = {
    id: query?.data?.length + 1,
    first_name: "Name",
    last_name: "Surname",
    email: "name-surname@example.com",
  };

  const handleSend = async () => {
    // mutation.mutate(toBePosted);
    try {
      await mutation.mutateAsync({ ...toBePosted, first_name: "zzzz" });
    } catch (error) {
      console.log(`Error in mutating asynchronously: ${error.message}`);
    }
  };

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  if (mutation.isLoading) return <h1>Sending data...</h1>;
  if (mutation.isError)
    return <h1>Error in sending data: {mutation.error.message}</h1>;

  return (
    <div>
      <button onClick={handleSend}>Send Data</button>
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

## Query results in pages

We can also have data in pages using the Tanstack Query. In our example,

- We first change our `fetchData` function that requests data from the first page of the url (json-server enables us to have pagination easily using the `_page=`).
- We add `page` as a variable to our query key, because query keys should be unique to properly identify the requested data from the server.
- The returned data from the json-server, will be different when we are requesting data in pages. The response will have info about the previous page in the `prev` property, next page in the `next` property, and the data that we are requesting will be in the `data` property. We are going to use this information to get the proper data and have functionality to change pages via buttons.

```js
// api/api.js
export const fetchData = (page) => {
  return new Promise((resolve) =>
    setTimeout(async () => {
      const response = await fetch(
        `http://localhost:8000/result?_page=${page}`,
      );

      if (!response.ok)
        return new Error(`Can't fetch data. Status: ${response.status}`);

      const data = await response.json();
      resolve(data.data);
    }, 1000),
  );
};
```

```js
// App.jsx
import { fetchData, addData } from "./api/api";
import { useMutation, useQuery } from "@tanstack/react-query";
import { queryClient } from "./main";
import { useState } from "react";

const App = () => {
  const [page, setPage] = useState(1);

  const query = useQuery({
    queryKey: ["users", page],
    queryFn: () => fetchData(page),
  });

  const mutation = useMutation({
    mutationFn: addData,
    onMutate: () => {
      console.log("mutation is starting");
    },
    onSuccess: () => {
      console.log("mutation was successful");
      queryClient.invalidateQueries(["users"]);
    },
    onError: () => {
      console.log("Error while mutating");
    },
    onSettled: () => {
      console.log("mutation ended");
    },
  });

  const toBePosted = {
    id: query?.data?.length + 1,
    first_name: "Name",
    last_name: "Surname",
    email: "name-surname@example.com",
  };

  const handleSend = async () => {
    mutation.mutate(toBePosted);
  };

  if (query.status === "loading") return <h1>Loading...</h1>;
  if (query.status === "error")
    return <h1>Error in fetching data: {query.error.message}</h1>;

  if (mutation.isLoading) return <h1>Sending data...</h1>;
  if (mutation.isError)
    return <h1>Error in sending data: {mutation.error.message}</h1>;

  return (
    <div>
      <button onClick={handleSend}>Send Data</button>

      <button
        onClick={() => setPage((currentPage) => Math.max(currentPage - 1, 1))}
      >
        Previous Page
      </button>
      <button onClick={() => setPage((currentPage) => currentPage + 1)}>
        Next Page
      </button>

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

## Query with infinite scrolling

Instead of having data in pages, we can have infinite scrolling using Tanstack Query. To implement this,

- we will import and use `"react-intersection-observer"` library. The `useInView` hook from this library will help us identify when an element is visible on user's screen.
- We will also use `useInfiniteQuery` hook from Tanstack Query. This hook has
  - `initialPageParam` which lets us provide the number for the first page of the data, and
  - `getNextPageParam` property which accepts a function that could help us to calculate what number we want as the next page.
- `useInfiniteQuery` returns `hasNextPage`, `isFetchingNextPage` booleans and `fetchNextPage` method all of which are useful to get the data for a new page.

```js
// App.jsx
import { useInfiniteQuery } from "@tanstack/react-query";
import { useEffect } from "react";
import "./App.css";
import TodoCard from "./components/TodoCard";
import { useInView } from "react-intersection-observer";

function App() {
  const { ref, inView } = useInView({});

  const fetchTodos = async ({ pageParam }) => {
    const res = await fetch(
      `https://jsonplaceholder.typicode.com/todos?_page=${pageParam}`,
    );
    return res.json();
  };

  const {
    data,
    status,
    error,
    fetchNextPage,
    isFetchingNextPage,
    hasNextPage,
  } = useInfiniteQuery({
    queryKey: ["todos"],
    queryFn: fetchTodos,
    initialPageParam: 1,
    getNextPageParam: (lastPage, allPages) => {
      // console.log({ lastPage, allPages });
      const nextPage = lastPage.length ? allPages.length + 1 : undefined;
      return nextPage;
    },
  });

  useEffect(() => {
    if (inView && hasNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, fetchNextPage]);

  if (status === "pending") return <p>Loading...</p>;
  if (status === "error") return <p>Error: {error.message}</p>;

  const content = data?.pages?.map((todos) => {
    return todos.map((todo, index) => {
      if (todos.length === index + 1) {
        return <TodoCard innerRef={ref} key={todo.id} todo={todo} />;
      }
      return <TodoCard key={todo.id} todo={todo} />;
    });
  });

  // console.log(data.pages);

  return (
    <div className="app">
      {content}
      {/* <button
        ref={ref}
        disabled={!hasNextPage || isFetchingNextPage}
        onClick={() => fetchNextPage()}
      >
        {isFetchingNextPage
          ? "Loading more..."
          : hasNextPage
          ? "Load more"
          : "Nothing more to load"}
      </button> */}
      <h3>Loading...</h3>
    </div>
  );
}

export default App;
```

```js
// TodoCard.jsx
const TodoCard = ({ todo, innerRef, ...props }) => {
  return (
    <p key={todo.id} ref={innerRef} className="todo-card" {...props}>
      {todo.title}
    </p>
  );
};

export default TodoCard;
```
