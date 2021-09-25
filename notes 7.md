/*
import React, { Component } from 'react';    
Importing the react library and the component class from react 

export default class Form extends Component {  
Giving all the methods that are defined in the react component class just how we're able to access props and state 


Forms
forms are the main places where you are using State because they're internal to the component and can be updated when the user interacts. Used to model things that users can change. In React, we pay attention to everything a user does as they make changes to the form in real time, and we update the form data as the User interacts with the form, and when it's time to submit we take the form data that we have been building the whole time and put it through. This allows us to do client-side validations in real time, and inform the user if they need to make corrections before they submit. 

constructor(props){
    super(props)
    this.state = { 

This is the only time we will see something like this. We don't use set_state within the constructor when we're initializing state for the first time but every time state changes later we use set_state.  This object is never modified directly. Because whenever the state changes we want react to be able to respond to the change and set_state is how we tell react that something has occured that it needs to take action on. 

Event Listeners With React  
Rather than having Event Listeners attached separatly they are attached inline with the JSX element they're going to be attached to. 

OnChange = {}
The OnChange Event Listener is going to be a function that takes an event as an argument. We're using an Arrow Function with this.setState() and we want the state not the text to change to whatever the contents of the text area. 
e.target.value will be the text area value. This ties the contents of the text area to state. 

*/