## Refactoring.

Sometimes we refactor code in anticipation of other features that will be added in a subsequent phase or possible future expansion. Currently our movies context's state is a simple array, and its mutation is straightforward, but this might grow in the future. Therefore, we will switch to the useReducer hook for managing the Discover movies array. In `src/contexts/movieContext.js` replace the entire code with the following:
~~~
import React, { useReducer, useEffect } from "react";
import { getMovies } from "../api/tmdb-api";

const reducer = (state, action) => {
  switch (action.type) {
    case "add-favorite":
      return {
        movies: state.movies.map((m) =>
          m.id === action.payload.movie.id ? { ...m, favorite: true } : m
        ),
      };
    case "remove-favorite":
      return {
        movies: state.movies.map((m) =>
          m.id === action.payload.movie.id ? { ...m, favorite: false } : m
        ),
      };
    case "load-discover-movies":
      return {
        movies: action.payload.movies,
      };
    default:
      return state;
  }
};

export const MoviesContext = React.createContext(null);

const MoviesContextProvider = (props) => {
  const [state, dispatch] = useReducer(reducer, { movies: [] });

  const addToFavorites = (movieId) => {
    const index = state.movies.map((m) => m.id).indexOf(movieId);
    dispatch({ type: "add-favorite", payload: { movie: state.movies[index] } });
  };

  const removeFromFavorites = (movieId) => {
    const index = state.movies.map((m) => m.id).indexOf(movieId);
    dispatch({
      type: "remove-favorite",
      payload: { movie: state.movies[index] },
    });
  };

  useEffect(() => {
    getMovies().then((movies) => {
      dispatch({ type: "load-discover-movies", payload: { movies } });
    });
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <MoviesContext.Provider
      value={{
        movies: state.movies,
        addToFavorites: addToFavorites,
        removeFromFavorites: removeFromFavorites,
      }}
    >
      {props.children}
    </MoviesContext.Provider>
  );
};

export default MoviesContextProvider;
~~~
Some points to note:

+ The state is now an object with one property: movies: [. . . ]
+ The useReducer hook takes two arguments:

    + The initial value for the state object, i.e.
{ movies: [] }
    + a function that manages the mutation of the state, termed a reducer.

+ useReducer returns a reference to the current state object and a reference to a dispatcher function. We use the dispatcher to send mutation requests to the reducer.
+ Our reducer function supports three types of state mutation:

    + Load the state with movies returned by TMDB Discover endpoint.
    + Tag a movie as a favorites by setting its favorite property to true.
    + Untag a favorite movie.

+ The dispatch function signature includes a mutation type string and the payload/data associated with the mutation. The mutation types are arbitrary.
+ Internally the dispatcher invokes the reducer, passing it the current state object and the mutation details.
+ The reducer does NOT change the current state object, instead, it returns a copy of it with the relevant changes applied. The copy becomes the new state object. This state-change strategy is mandated by the useReducer hook.

In the browser, check the add and remove favorite features works as before.

Update the project git repository:
~~~
$ git add -A
$ git commit -m "Refactor movies context to use useReducer hook"
$ git push origin main
~~~