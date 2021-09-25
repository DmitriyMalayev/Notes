`yarn install`
After opening a lab make sure to run yarn install
yarn is a dependencies manager similar to npm 

`yarn start`
To run in the browser type yarn start 



# Final Review Question 
# What is the difference between functional and class based components?
`Class Based Components`
Have life cycle methods like render and componentDidMount() componentWillMount() componentDidUnmount() shouldComponentUpdate()  
They run in the life cycle of a component 
Has the ability to mount and it can re-render itself because it has state 
It also has a sequence of methods that get called that are managing how we actually update the DOM rather than just calling the function getting the JSX and putting it in there. 
`this.` is necessary because we have multiple methods that are getting called and we still want access to information related to the particular component that we're working with because we might have more than once instance of the comment component like multiple comments that are getting rendered. 
When you make a new comment the constructor is getting invoked but when you render it that's another method. 
If you need access to the arguments that were passed through the constructor they have to be stored somewhere and that somewhere is in the `this.` object, which is the properties of the object that you're calling render on.   

When you make a new instance of a class based component you have the constructor get called. 
The constructor is assigning properties to `this.` 

Render is another method that in order to access information it needs to access the object that you're calling it on which is `this.`

`Functional Components`
Functions that return JSX 

The props that you get in a functional component are just an argument that you pass whereas the props in a class based component are assigned in the constructor, and they are accessible in connection with the component object itself this is why `this.` is required to access the props in a class based component. 


# Notes 
Every time you update a form you're updating the state of the component which allows you to do things like display validation error messages as you go rather than waiting until someone is done. When you submit, the state is passed in. 

Data determines what's in the DOM and what the user sees.  
Rather than pulling data out of the DOM to figure out what to send as a fetch request. We are having the event listeners that we attach to user behaviours update your data and state, and then that update is going to trigger the DOM updates. 

`Data, Behavior and Display`
Data and Display work together  
Whenever the Data changes the Display updates 

Behavior and Data work together  










