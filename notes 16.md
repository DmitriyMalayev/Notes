# npx create-react-app todo-app-react  
This provides the most recent version of the react app generator 
In comparison with npm install -g   (CHECK)

# package.json 
javascript equivalent of a gemfile 


```json 
{   
  "name": "todo-app-react",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.11.4",
    "@testing-library/react": "^11.1.0",
    "@testing-library/user-event": "^12.1.10",
    "react": "^17.0.1",
    "react-dom": "^17.0.1",
    "react-scripts": "4.0.1",  //has many sub-dependencies bundled into it 
    "web-vitals": "^0.2.4"
  },
  "scripts": {  
    "start": "react-scripts start",  //allows you to view your app in the browser and see changes live 
    "build": "react-scripts build",  //creates a production build version of the app without live reloading 
    "test": "react-scripts test",  //runs tests if configured  
    "eject": "react-scripts eject"  //avoid 
   },
  "eslintConfig": {
    "extends": [  
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {  // How code we write turns it to things that run in the browser 
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}

```

`Props` 
External data that are passed on from the parent to the child 
Passing props from one component to another is done via something that looks like an HTML attribute within the JSX


```js 
import TodoList from "./todo_list";
function TodoLists() {
  return (
    <>
      <h1> TodoLists Component</h1>
      <ul>
        <TodoList name="my name has changed" />
      </ul>
    </>
  );
}
export default TodoLists;
``` 


This allows access in the child 
```js 
function TodoList(props){
    return (
      <li className="my-2 px-4 bg-green-200 grid grid-cols-12 sm:grid-cols-6">
        <span className="py-4 col-span-10 sm:col-span-4">{props.name}</span>
        <span className="my-4 text-right"><i className="fa fa-pencil-alt"></i> </span>
        <span className="my-4 text-right"><i className="fa fa-trash-alt"></i> </span>
      </li>
    );
}
export default TodoList; 
``` 

You'll see destructuring frequently with React Apps, particularly in Functional Components. 
In our example, what we would do is: 

```js 
/*
We can add another prop with a comma.
We're pulling out the value of the property in the object and storing it in the variable called name. If there is no such property then name will be undefined within the function.  
*/  


function TodoList({name}){ 
    return (
      <li className="my-2 px-4 bg-green-200 grid grid-cols-12 sm:grid-cols-6">
        <span className="py-4 col-span-10 sm:col-span-4">{name}</span>
        <span className="my-4 text-right"><i className="fa fa-pencil-alt"></i> </span>
        <span className="my-4 text-right"><i className="fa fa-trash-alt"></i> </span>
      </li>
    );
}
export default TodoList; 
``` 




 # FINAL REVIEW QUESTION
 `What is the difference between State and Props`
 
 `State` 
 Internal 
 Local variable that changes the state of the UI of a component (whether a checkbox is checked indicating whether we want to see completed tasks)
 This is something that user has to be able to change and interact with in some way
 With react, when something important changes react reacts to this change by automatically updating the DOM for us. 
 It should only be able to update in State.
 The State in one component can be passed in as a Prop to another component. The prop component that's receiving the state component should not be able to change the state directly 
  
`Props`
External (From Parent)
Arguments passed from one component to another as an argument  
Static within a component (doesn't change)



# Tree Of Nodes 
`Data Code`
State 
Props 

`Behavior Code`
Event Listeners  

`Display` 
return 
JSX 









