# Mutations

- [Mutations](#mutations)
  - [`useMutation`](#usemutation)
  - [`useMutation` Callback](#usemutation-callback)
  - [Updating The Query After Mutation](#updating-the-query-after-mutation)
    - [Using `setQueryData`](#using-setquerydata)
    - [Using `invalidateQueries`](#using-invalidatequeries)
  - [The `mutateAsync` Method](#the-mutateasync-method)

---

---

## `useMutation`

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

---

---

## `useMutation` Callback

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

---

---

## Updating The Query After Mutation

### Using `setQueryData`

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

---

### Using `invalidateQueries`

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

---

---

## The `mutateAsync` Method

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

---

---
