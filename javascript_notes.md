`document.querySelector();` 
The method is querySelector 
The method is said to be "defined in" the object `document`

It takes in an argument in the form os a string. 
It also expects us to provide a CSS identifier that will help us find the node we want as an argument. 
If the document object finds an HTMLElement in it's representation of the page, it will return it. 
If it does not find it, it will return null. 

```js
document.querySelector("div") 
<div> class="position-relative js-header-wrapper" </div>     

document.querySelector("not-div")
null 
``` 

The thing document.querySelector() returns is also an object. It also has info, methods, state, behavior and properties. 
The HTMLElement instance has methods like remove() 



```js
document.querySelector(selector) 
``` 

`selector`
The selector is like a query string that let's us find things within an HTML page. It's like how SQL finds records in a DB. 
What is the syntax of this selector? 
How does the selector navigate through our document to find the DOM nodes that we want to work with (update, move, delete, etc.) 







```js 
er = World.create({name: "Earth"}); 
>> World {name: "Earth"};

// debugger below constructor 

attributes 
>> {name: "Earth"};

Object.keys(attributes)
>> ["name"]

this
>> World{} 

this["name"] = attributes["name"];  

this 
World {name: "Earth"} 


World.create({name: "Asgard"})

this
>> World {}
//In the constructor this refers to the object we're building   

this.constructor()
>> class World extends CRUD {} 
//this.constructor refers to the function that we would use to build this object which is the same as the class. 

this //Outside of Debugger 
>> Window {window: Window, self: Window, document: Document, name: "", location: Location, ...}

World.constructor()  //This is a function. In JS, all functions are classes.  
>> f anonymous () {}  

function World(){}  //Another way of writing the above function. 

// Basically you're creating a model object. A function with an uppercase name is one that you're you're designing to be like a a builder of new objects. You can call new, then the name of the function and then pass in an argument and that argument is the one that gets passed to the the function which is constructing a new object so in this case World is a constructor function but in in the end is a function. 
// When you call constructor on a class you're gonna get a function. This is important because and this is again one of the trickiest parts of JS but the one of the key concepts is this to understand that "the only thing that creates a new context in JS is a function" Every time you define a class the reason you're able to use this and have it be different and have it change depending on context is that the class is a function and functions allow you to create a new context. 
// When you create a new instance of a class any method you call on that instance will have the context of that instance. The this keyword will refer to that object you're calling the method on and if you call a method directly on a class via one of the static methods "this" inside of that method is going to refer to the class that you're calling the method on. 

// This part does behave a lot more consistently like you're used to with ruby it's when you start pulling in Arrow functions that things get weird because they they don't have their own context. If we needed to ensure that we could access the same context as we're currently in within a function using an arrow function would help us to do that because the arrow function doesn't create a new context whereas creating a class would. If there's something in the "this" object that we want that we need to use using an arrow function we'll make sure we're able to do that. 

// Our code's purpose is to allow you to pass in an object and then use it to build a new instance of the class and then save it so that the class can remember it right so this is similar to before we had active record right this might have been how we would we could have done this in like the music library CLI all the way back there right if you call create pass in some attributes they get assigned you bank a new instance and you've added to @@all. 
// what we're doing here is we're calling a collection instead of all because if we called it all we would actually be referring back to the same method so what we're doing is we're defining a property called collection on "this" which again is what inside of a static method the class right. So this is going to be the class that we're calling all on and we're assigning a property called collection equal to either whatever it already is or if it doesn't exist yet an empty array. @@all ||= [] 

// static methods in JavaScript are how we implement class methods. Similar to def self.all in Ruby. 
// static all() {} is equivalent to @@all ||=[]  
// We use this.collection instead of this.all because "this" is the class and this.all is actually this method we're defining, because of this we can't assign it to an array. We are storing all of the instances of the class in a property of the class and allowing access to it via a class method .all()

// The static create(attributes) method takes an object filled with attributes as an argument and uses them to create a new instance of the class. It adds that instance to the collection of all saved instances and returns the new instance. 


// then find by this method takes an object filled with attributes as an argument uses them to find an object in in this.all that has all of those attributes or matches say all of those and then find by id pretty straightforward this method takes an id is an argument and finds the object in just at all that has that maybe and we can also add turns undefined if nothing if no such object exists so this again turns undefined if no such object exists and finally update this method takes an object filled with attributes as an argument uses them to update the object's attributes with those values so so in this case this method might be called after it might be called on the return value of find buy id right so we might have some kind of an update that we do refined it by an idea then we we call update on it to change the value 


```














