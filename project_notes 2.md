`Actions`
    Actions are passed to reducers
`Reducers`
    Reducers can update the state
    Only reducers can do this.
`has_one_attached`
    Specifies the relationship between a single attachment and the model
`Rails.application.routes.url_helpers.url_for(recipe_image) if recipe_image.attached?`
    Same as self.recipe_image.attached 
    We're able to use shorthand due to has_one_attached implementation 
    We're adding a method to the Recipe model which will allow us to fetch the recipe image to the recipe in the api.   
    We also adjust the RecipeSerializer and default_url_options in development.rb
  

```js
// REFERENCE 
const mapStateToProps = (state) => {
  return {
    cuisines: state.cuisines.list,
  };
};
const mapDispatchToProps = (dispatch) => {
  return {
    dispatchFetchCuisines: () => dispatch(fetchCuisines()),
    dispatchCreateRecipe: (recipe) => dispatch(createRecipe(recipe)),
  };
};
export default connect(mapStateToProps, mapDispatchToProps)(NewRecipeContainer);
```

`mapStateToProps`
    Subscriber 
    Provide data and access to updates so it can to update to the data.   ?? 
    Does not provide the ability to make changes, just to be able to receive updates made by others.
`mapDispatchToProps`
    Publisher 
    mapDispatchToProps adds function(s) to props that will dispatch the return value of an action creator.
    Provides functions that would publish to all subscribers. 
    Provides the ability to make new issues for all subscribers.
`What starts the redux process?`
    Dispatch 
`componentDidMount()`
    Called immediately after a component is mounted. Setting state here will trigger re-rendering.
`import { connect } from "react-redux"`
    connects react and redux
`import React, { Component } from "react"`
    Importing modules and methods from the react library as a component ?? 
`import { Link } from "react-router-dom"` 
    Allows us to add links that we can convert to routes
`Named Export vs. Default Export`
    If we're importing a named export, we have to use curly brackets {}
    If we're importing a default export, we do not need curly brackets
`Root Reducer`
  Redux uses a single reducer function that accepts the current state and an action input and returns a new state. 
  The first time it runs it doesn't have a value, this is why we make default set to initState. ??

```js
const rootReducer = combineReducers({
  auth: authReducer,
  project: projectReducer,
  //We're specifying which reducers we want to combine together and what do we want to call each individual reducer
  //The auth reducer will update the auth state object
  //The project reducer will update the project state object
});
```



`filter`
  We can use the filter method because it is non destructive. 
  It assigns a new value, same things are kept, others are removed. 
`connect`
  Connect is a function that returns a higher order component so it can connect to the Redux Store. 
  We need to be able to retrieve data from the store. 
  If a component wants access to the store, we wake some data from the store and we map that data to the props of our component.
  When we connect to redux, it knows what data it wants to grab from redux, and the property we want to create from our props object to apply that data to. ?? 




/*
  mapDispatchToProps
    If we want to interact or make a change to the state from this component we have to be able to dispatch an action from the component. The action will contain a Type and an optional Payload, the payload can be the id of the post we want to delete. The action is dispatched to the reducer, and the reducer takes the action and checks the type of action and takes in the payload (id) we want to delete from the state and it makes the change from the central state. When that changes we get the updated props inside of the component.  mapDispatchToProps takes in a parameter which is the dispatch method.
  
  REFERENCE   
    We don't need to use store when calling dispatch
    store.dispatch({ type: "ADD_CUISINE", cuisine: "Italian" });  
  
  return
    represents what properties or what functions we're going to map to the props of this component.
  deletePost 
    We want to map a function called deletePost, that will dispatch an action with a type and id, which is passed as an argument in deletePost. 
    Whenever we call the function deletePost we are dispatching { dispatch({type: 'DELETE_POST', id: id}) }
    deletePost is going to be attached to our props, so we can use it inside of our component

  mapStateToProps    
    We want to grab a single individual record. We can use ownProps, it refers to the props of the component before we attach the additional props from the Redux Store.  We use post_id because it's how we reference it when we set up the routes in App.js
LONG VERSION
  const mapStateToProps = (state, ownProps) => {
    let id = ownProps.match.params.post_id
    return {
      post: state.posts.find((post) => { return post.id === id})
    }
  const mapDispatchToProps = (dispatch) => {
    return {
      deletePost: (id) => {
        dispatch({ type: "DELETE_POST", id: id });
      },
    };
  };
  */




/*
export const createProject = (project) => {     WITH THUNK
	return (dispatch, getState) => {};
};
dispatch => Dispatches an action to the reducer. When we first call an action creator inside of a dispatch method from our component we're returning a function and pausing the dispatch because we're not returning an action anymore we are returning a function. 
We first make an async call to the DB and then proceed with dispatch. 

getState => Used if we need to retrieve the state


 export const createProject = (project) => {   BEFORE THUNK
 	return {
 		type: "ADD_PROJECT",
 		project: project,
 	};
 };


dispatch(action={type: payload:})
reducer(state, action) 

function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => (next) => (action) => {   //next is always dispatch for us 
    if (typeof action === "function") {    typeof action refers to asking if action is a function 
      return action(dispatch, getState, extraArgument);   returning action
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;

import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import registerServiceWorker from "./registerServiceWorker";
import { createStore, applyMiddleware } from "redux";
//createStore and applyMiddleware are both functions

import rootReducer from "./store/reducers/rootReducer";
//Only a reducer can update the state. rootReducer contains combineReducers.

import { Provider } from "react-redux";
//Importing the Provider component which can surround our app and pass the store into the application. This will provide the application access to the store.

import thunk from "redux-thunk";

const store = createStore(rootReducer, applyMiddleware(thunk));
/*
Takes in a rootReducer so it has something that can be used to update the store.
The rootReducer contains the combineReducers function

applyMiddleware() 
This function is a second argument of createStore()
It can take in a list of middleware and it turns into a store enhancer so we could add many different middlewares inside. We could also have many different store enhancers and they will enhance the store functionality.

We apply thunk to the middleware as an argument. 
thunk will enhance the functionality of our store. 

Thunk
thunk allows us to return a function inside of our action creators which can then interact with a database.
*/

ReactDOM.render(
	<Provider store={store}>
		<App /> {/* Providing the application access to the store*/}
	</Provider>,
	document.getElementById("root")
);
registerServiceWorker();

//createStore can accept one reducer or a rootReducer which is a collection of many reducers. We are able to create multiple reducers for different parts of our app each one handling their own set of actions.


`CHECK??`
    This creates a new recipe. The inputs in the form need to match the schema.
    For our recipe_image we implement has_one_attached :recipe_image.
    It uses ActiveStorage

    We need to add the Data from the Form into this body (FormData) object using the append method. When we do, we want to be thinking about how the rails API is expecting the event_params to look.
    We're pulling out data from out of the target of our submit event and attach it to a formData object we're building so we can send that as a body of our post request to create the new event.

    We add a link from the CuisineShowContainer to the route where we can add a recipe to the cuisine. ??
