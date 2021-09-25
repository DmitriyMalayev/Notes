# DOM CHALLENGE OUTLINE
A Counter that increases by 1 each second 
Plus and Minus buttons that increment or decrement the counter
A "like" button (❤️) that adds a "like" for the number that is currently displayed by the timer. 
A comment box that adds comments when submitted 

# AS A USER
I should see the timer increment every second once the page has loaded
I can manually increment and decrament the counter using the plus and minus buttons
I can "like" an individual number of the counter. 
I should see a count of the number of "likes" associated with that number. 
`I can pause the counter which should`
Pause the counter 
Disable all buttons except the pause button
`The pause button should then show the text "resume."` 
When "resume" is clicked, it should restart the counter and re-enable the buttons. 
I can leave comments on my gameplay, such as: "Wow, what a fun game this is." 
  
# DATA
counter(integer)
likesCounter(object with keys of numbers and values of how many likes they have) 
  {1: 2, 2:0, 3:3} 
comments (array of strings)  

# DISPLAY (What pieces of html will house our data? These are 3 DOM Nodes)
`h1#counter => counter` This displays the actual counter  
`ul.likes => likesCounter` ul has the class of Likes 
`div#list => comments`
<li>1: 1 </li>   If we click like on number 1, it will show we liked 1
    

# BEHAVIOR (Events)
`DOMContentLoaded => attachListeners, start counter`   (This is the initial page load event) 
click on plus
click on minus
click on heart
click on pause
click on resume
submit comment form

# UPDATE DATA 
incrementCounter() 
decrementCounter() 
increaseLikes()  Needs a number when we click on like. This could be implemented with an argument or another way. ??
pause()
resume()
addComment() 

# DISPLAY NEW DATA BY DOM MANIPULATION  

`When we click on`
plus we call incrementCounter(), and update the counter h1.innerText 
minus we call decrementCounter(), and update the counter h1.innerText 
heart we call increaseLikes(), and update our list of  likes ul.likes
pause we call pause(), and disable all of the buttons (except pause) and relabel the button to resume 
resume we call resume(), reenable all of the buttons and relabel the button pause 
`When we submit the add comment form` 
We call addComment() and append the comment to div#list 





`.constructor` 
Refers to the prototype function that was used to build an instance or object 
JavaScript has protypal inheritence 

`Whenever we need to count a group of things think of the object as a data structure`
Keys => refer to the things you're counting  
Values => refer to how many you have found of that thing 

`The reason you want to use an object for likes`
We need to store information separately in different places depending on when we click on the heart button. 
In comparison, when you submit the comment form this isn't going to change what you do, or where the comment will be submitted. 
We use an array of strings for comments. 

`likesCounter(object with keys of numbers and values of how many likes they have)`  
  {1: 2, 2:0, 3:3}

`setInterval()`
The setInterval() method, offered on the Window and Worker interfaces, repeatedly calls a function or executes a code snippet, with a fixed time delay between each call. It returns an interval ID which uniquely identifies the interval, so you can remove it later by calling clearInterval()
This method is defined by the WindoworWorkerGlobalScope mixin. This is an asynchronous function.  
`Syntax` 
var intervalID = scope.setInterval(function[, delay]) 



The reason things aren't as predictable in JS as it is in Ruby 
Functions in JS are first class objects 
In Ruby once you defined a method the only thing that you can do with it is call it 
In JS when you define a function that function itself is an object. 
You can add properties to it, you can pass it as an argument to another function, you can store it as a property of another object. Because of this we cannot assume how a function will be called. 
`this` allows us to refer to the context in which the function was called 
If you call an instance method on an instance of a class, `this` would be that instance. 
If you store `this` function, you store the `start` method for class Counter somewhere else in your code and then you invoke it wherever you stored it `this` will be different. 

`Functions in JS can be bound`
We can modify what `this` will be directly in your code by using 3 methods 
1. Bind
2. Apply 
3. Call 

Apply and Call are similar with different syntax. They call a function and allow you to specify what the `this` value should be
Bind returns another function with the context bound to whatever you specify. 

```js
start() {
    this.interval = setInterval(this.incrementCounter.bind(this), 1000)
    }
``` 
This is saying the callback that we're passing to set interval, the function that we want to get called every 1000ms we want it's context to be bound to the counter so that it has access to this.counter 
If we didn't do this, the `this` inside of increment counter would not be the same `this` that has the counter 
```js
start(){
    this.interval = setInterval(() => this.incrementCounter(), 1000) 
    }
```
One of the fundamental differences between Arrow Functions and Regular Functions. (Important to know for JS and React)
Arrow Functions don't have their own context. What `this` is originally, stays the same. 
Bind does not work on Arrow Functions 
The only thing that creates a new value for `this` in JS is a function keyword function 
Classes under the hood are function keyword function 

With Regular Functions context is set based on how it gets called 
With Arrow Functions context is set at definition time. `this` is determined when defined. 


`Element.matches`
Returns trye if matching selectors against element's root yields element, and false otherwise 


`EventListeners and Classes`
The EventListeners determine what happens with which objects when events happen 
The Classes are managing how to do it. How to update the DOM that corresponds to that object 