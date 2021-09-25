`Container` 
Manages behavior and data.  
(RAILS CONTROLLER) (Comparison) 

`Presentational Components`  
Focused on Data and Display.  ??        
Presentational Components focused mainly on Data and Behavior and their rendering logic is mainly about rendering components and those other components are the ones that manage Data and Display. They take in props and they render JSX.     
(ERB VIEW) (Comparison) 

`NavBar.js` 
<nav className={`navbar ${colors[props.color]}`}>   
Whatever color is specified in props is going to be applied to this nav class. Uses bootstrap classes.  


`GifListContainer` => Behavior & Data
`Container` => Behavior & Data 
`Presentational` => Data & Display 


`Behaviors`
Type in search input 
Submit search form 

`Data` 
Content of search form 
Gifs loaded from API

`Display`
NavBar 
Form
Images 


`Components`
App 
GifListContainer 
GifList 
GifSearch 
NavBar 


# GifSearch

`Behaviors`
Type in search input
Submit search form 
We will need an event handler but it will also need to update state of it's parent component with a function that is passed down as a prop)

`Data` 
Content of search input 

`Display`
Form

`Notes`
is the submit going to be something we care about from GifSearch or it's mainly relevant to the container. Does the behavior of Behavior affect Data?  No it does not. 





# GifList

`Data` 
Gifs loaded from API (props) 
Props are used 


`Behaviors`
No user actions take place here. 
It's a Presentational Component. 

`Display`
Images in List

`Notes`
Just because User Actions take place within a component, where you would need to attach an event handler that doesn't mean that a component is not presentational because it could be a callback. You can pass a callback down to a presentation component and have an event listener in that component that will the callback that's passed down, but the function itself, the behavior, is managed in data somewhere else. The container component for the presentational component can pass down a callback that updates the container state to a child component that is presentational. That child component just takes in props and renders JSX it might have attached listeners but it doesn't contain the logic for what happens inside the component itself. 


# GifListContainer

`Data` 
Gifs loaded from API (State)

`Behaviors`
submit search form and use the input from the user as a search query => Hit API and setState() 
GiflistContainer stores the data that we get from the API and also getting that data and storing it in state. 
Handles the search and updating the state accordingly. 

`Display`
GifSearch(pass in the callback to handle search) 
render the GifList(images from state)
The render method is rendering the other components (GifSearch and GifList) this is what allows us build what we see in the demo. 

`Notes`
GifListContainer is the Parent

The data that handles behavior is coming from the container, in GifSearch some behaviors occur as well (typing, submit which needs a preventDefault and a response). The logic that makes our program work and that decides what happens when the submit button is clicked is in the GifListContainer. 


`Super Parent`
App => Renders NavBar and GifListContainer 

`Parent`
GifListContainer 

`Children`
GifList
GifSearch 