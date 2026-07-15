# Tanstack Query (formerly React Query) notes

<hr>

- [Tanstack Query (formerly React Query) notes](#tanstack-query-formerly-react-query-notes)
  - [Some useful links](#some-useful-links)
  - [Getting Started](#getting-started)
  - [Queries](#queries)
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

## Getting Started

[Getting Started](./TS_GettingStarted.md)

<hr>

## Queries

[Queries](./TS_Queries.md)

<hr>

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
