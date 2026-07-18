## Query Results in Pages

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

---

---
