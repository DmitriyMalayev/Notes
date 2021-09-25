The ProductView class has a static method called render (static is analagous to a class method in Ruby)
When we call the render method and pass in a product object, it will return a div element with some html.

`class ProductsList`
The ProductsList class allows us to keep track of a list of products and the container on the page that we will display them in. 
When we create a new instance of this class, we will attach a particular DOM element to it. 
We can also create a list of products and pass that in next. 

The constructor here is analogous(similar) to the initialize method in a Ruby class.
The class has a method addProduct() to allow adding a product to the list. 
This method will actually assign an id to the product, add it to the list of products, and incremenent the id.
It increments the id for assigning a new id to the next product. 
The method also appends an element to the container to display the product we just added. 

This is analagous(similar) to self in ruby, except for the fact that we can't use it implicitly. 
In Ruby, if we skip invoking a method, it interprets what we did as we did self.name_of_method
In JS, if we want to access a property stored inside of this, we have to do this. ?? 

When we do this.container = container, this is similar to @container = container in initialize. 
In JS we don't need to define a setter and getter to be able to access information in this object. 
If a property exists in the object we can access it. Similar to console.dir()


`addProduct()`
This can receive a product object which is similar to a Ruby Hash. 


`ProductView Class`
The ProductView Class has a statis method called `render` 
A Static Method in JS is similar to a class method in Ruby

We do not have to create an instance first when we call the render method??.
When we call the render method and pass in a product object, it will return a div element with some HTML info about the product inside. This is similar to self in Ruby, except for the fact that you can't use it implicitly. 

`render() {}`
Our render method contains HTML 

`Single Responsibility Principle`
With the ProductView it's just is just how do we display things. 
It does not need to know how to track. 

`new ProductsList(cont, products)`
When we invoke ProductList we pass in the container and products 
Then this is going to attach it to the element with an id of "#productsContainer"


`Padding vs. Margin`
If you're in a case where you want to allow a background image go to the edge of the creen you should use padding. With margin it will not. ?? 

If we apply the padding to the body then everything inside of the body will be starting away from the edge. Because of this we will not be able to have a full background. 

`What’s the Difference Between Margin and Padding in CSS?` 
Basically, a margin is the space around an element and padding refers to the space between an element and the content inside it. 

The margin falls outside two adjacent elements. Each side of the element has a margin size you can change individually. In creating the gap, the margin pushes adjacent elements away.

On the other hand, padding is placed inside the border of an element. To create the gap, the padding either grows the element’s size or shrinks the content inside. By default, the size of the element increases.



`<head> <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet"> </head>`
In order to use tailwind css make sure that line is on top of your index.html file. 
If you want to use this long term, use the NPM Package. 


`CSS Breakpoints`
CSS breakpoints are points where the website content responds according to the device width, allowing you to show the best possible layout to the user. CSS breakpoints are also called media query breakpoints, as they are used with media query. 

`Putting Code In Function`
We should put code inside of a function and then only call it when the DOM has loaded. 
One of the base events that we want to be aware of is DOM Content Loaded. It's important not to have JS running until your webpage finished loading.  

`DOMContentLoaded`
Fired when the document has been completely loaded and parsed, without waiting for stylesheets, images, and subframes to finish loading. We can attach an event listener to this and then set up your initialization code in the init function below.  

`function init() {}`
We wrap our code in this function. 




```js
console.log("A: before the event listener");

window.addEventListener("DOMContentLoaded", function () {
  init();
  console.log("B: DOMContentLoaded");
  attachListeners();
});

console.log("C: after the event listener");
```
`What order will we see the logs in?` 
1. A B C
2. A C B  THIS ONE
3. C A B

Every time you attach an event handler an event listener that function is async. 
All Event Handlers are Asynchronous because the browser is keeping track of events. 
No event happens immediately.  
The event handler callbacks will not be triggered until all Synchronous Code (function we have already code and code that's still running) has finished loading. 

Every event handler is asynchronous - the event handler callbacks won't be triggered until
all synchronous code (functions we've already called and code that's still running) has finished running.

An asynchronous function is one that's not called immediately but sometime later. It is hooked into some browser API and that takes a certain amount of time. There is a timeout (wait before the function is called) or it's an event handler and we wait for the event to happen and then we call the function or it's a callback attached to like an API for fetching data. 

Whether it's a fetch or http request that object that you're sending a request to a server and you're waiting for a response and then you cal a function anytime you have a function that's not invoked immediately that function is asynchronous and it will not run until all of the synchronous code is done. 

When our script is loaded the browser is going to run our JS and it's going to add the addEventListener for DOMContentLoaded and then log "c option" and then when the code is done running, this even will be registered like that it happened and then this function will happen right. The function handles that even that happens but no event handler gets invoked until all the synchronous code is done. 

Callbacks are asynchronous when they are the ones that do not get called immediately. 

`this.products.forEach(this.addProduct.bind(this))`
The function that we pass to forEach is invoked immediately for every element in the array before we move on.  

`What makes something asynchronous?`
The thing that makes something asynchronous is how it gets called. 
If it's going to be called in response to some asynchronous api like the fetch api, or set timeout, it's the browser's counting time and then when the time elapses it's going to call the function or if it's an event handler right that event handler won't get triggered until the browser passes that event to it and it's not going to do that until it's done rendering the page and reloading all of the JS. 

If we are attaching event listeners to the document it would probably be a good idea to wait until DOMContentLoaded goes through.

`attachListeners`
We can have a function called attach listeners that we call and then we define it above. 
Inside of this function we add different kinds of events. Such as adding an EventListener for click and submit. 
We're using event delegation here because when an event happens somewhere in the DOM that element that was targeted, is going to be the target of th eevent that's passed to the event handler. 
Also, the event will propogate to the top of the DOM tree unless it's captured somewhere and stopped. 

If we have a submit handler attached to this form, what we want to do is target it with an id of "addProduct" in index.html 
```html 
<form id="addProduct" class="mt-8 space-y-6" action="" method="POST"> 
``` 
The original target of the submit event is the form. But, the event will also be passed up to it's parent tags <div> </div> <body></body>  unless we stop it. 

`stopPropogation`
We can stop propagation using this function?? 

We can add a click even listener to the header for the product. 

```js
div.addEventListener('click', function(e))
console.log(`clicked on a product with id: ${e.target.dataset.id}`)  
``` 

We added a click event listener to each one of the product view divs and what we could do is set this with a data id.  
div.dataset.id is equal to the product.id 
We have a data id attribute in this div so this is going to be a common technique we will use. 
This allows us to have access to information that is attached to a particular part of the page. 


`e.target`
This is the target of the event. If we attach this event listener to the div, then the target is going to be the div that we clicked on. 

`this`
this is actually the div 
Within the handler if this is a regular function, not an arrow function it's going to refer to the object that you attach the event listen to. 

Instead of `e.target.dataset.id` we could use `this.dataset.id`

`The difference between Arrow Functions and Regular Functions`


Find Out?? 




`data attribute`
What we're saying is this header has the product id as a data attribute and we're saying when you click on something that has the class of product logged to the console the data attribute of that target elements data id value. If we click on Zevia we see this product has an id of one if we click on bean fields this product has an id of two but if you click on price or the description it it doesn't trigger that. 
The the trick there what we've done is we've basically said we this one element this one piece of the DOM is the one that we want to attach an event listener to and we're going to add a data attribute to that element so when that event happens we'll have access to some data that's related to the event. 
We attach to the html we're going to interact with some information that we're going to want to have access to when that event happens right so for example if we were gonna get additional product information when we clicked on this right uh or maybe like an image we wanted to fetch and have it show up and like like a modal come up when we do that we would need to know the id of the product to be able to make that request to our api to have access to that information. 

`This is how we can do it` 
We attach the when we actually put the html into the dom we attach a data attribute to the element that the user is going to interact with so that when they interact with it we have access to that data and we can use it to make the next request to do what the next thing we need to do is. 

`Project Info`
For your project then again you don't have to be worried about that now but for the for the project these would actually be instances of another class right you might have another class called product that had these attributes and then maybe had behavior attached like you could add it to your cart, like that or you could like it say if you like a product that would be updating its likes property. When you when you click on the uh the like button you have a CSS selector for a like button right something like if the target matches like button etc. You would have a data attribute for the product id, and use that use target data set id to access it find the product in the products list that has that id and update the likes property by adding one to it. 

`Submit`
Submit right the act of submitting you always want to prevent the default behavior which would be a page refresh and in this case again the way we're thinking of this our add product form but we might have more than one form on the page. What we want to do is every submit event that happens will propagate all the way to the document and then what we do is we determine we can still do this target thing and then you can do if target matches add product right so basically if the target of the submit event which is always going to be the form itself, if that matches add product then we handle this the way that we want to so we're going to say if it does.  


`manually pulling values out of inputs with e.target.querySelector("#name")`
if we actually need to pull out the values of these inputs we're probably going to have to do it with a query selector.
you can call querySelector on a document but you can also call it on any subset of the document so you don't have to look through the whole thing and in this case we know all the inputs are going to be inside of the form so we can do: 
if we look inside of the form what what's the first input that we want how do we find this name 

`console.dir(e.target.querySelector("#name"))`  
If you forget how to get what's inside the input how would you figure it out what's the helper you can type into the browser console to see everything inside of that object? We can use console.dir and pass in the argument.  
```js 
e.target.querySelector("#name").value  
=> "Jose"

e.target.querySelector("#price").value  
=> "19.99"

e.target.querySelector("#description").value  
=> "Date night with Jose for a good cause" 
```

`form data`
What we want in our our form submit event handler is to take the form data and build an object out of it so we can do 
the following. 
```js 
let product = {
  name: e.target.querySelector("#name").value,
  price: e.target.querySelector("#price").value,
  description: e.target.querySelector("#description").value
} 
```
`JavaScript Class Methods`
start with "static"

`Ruby Class Methods`
start with self  

# Stack Overflow
Imagine, you have a class Person. An instance of that class could be daniel.
`Instance method`
You could call a method on daniel, e.g: daniel.talk(), so daniel starts talking...
`Static method` 
You could call an static method on class Person, instead of on a concrete instance, for example: Person.getPeopleFromNewYork(). Note that getPeopleFromNewYork is not related to any instance of Person but to the class itself.
A method like getPeopleFromNewYork() would belong to some kind of repository, but it's just for the example.
