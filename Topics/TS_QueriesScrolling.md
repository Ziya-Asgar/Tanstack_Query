# Query with Infinite Scrolling

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

---

---
