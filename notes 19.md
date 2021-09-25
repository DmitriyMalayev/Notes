# JSON SERVER 
Install JSON Server

npm install -g json-server

create a file called `db.json`
  
```json    
//Make sure to use double quotes for consistency 
{
  "products": [
    {
      "id": 1,
      "name": "Zevia",
      "price": 4.99,
      "description": "Awesome soda with no sugar"
    },
    {
      "id": 2,
      "name": "Beanfields chips",
      "price": 3.69,
      "description": "Awesome gluten free bean chips"
    }
  ]
}
```

After we have a `db.json` file
We can use this file to create a RESTFUL API by running 

`json-server --watch db.json`
watch watches the file for any changes and if there are any the API will respond 
If we update the file it will update our running server as well 

`fetch`
returns a type of object called a promise

`promise`
A promise is a mechanism that we have for handling asynchronous code // code we want to run asynchronously 

`asynchronous callbacks`
Happen after all of the synchronous code completes 


```javascript

fetch('http://example.com/movies.json')
  .then(response => response.json())
  .then(data => console.log(data)); 
```

`.then` 
If you see this, it means what's right before it is a promise 
The .then method gets called on a promise object 
The .then method also returns a promise object 

`Note`
If we're inside of a promised callback the thing you're receiving as an argument is whatever the previous promise resolved with. 
If we're in a promised chain each then callback has to return a value that's passed on to the next then callback 






Start JSON Server

`json-server --watch db.json`   
Gives you the ability to make requests and the successful requests update your db.json file 
We're mocking a RESTFUL API by using a file and this package 

Now if you go to http://localhost:3000/posts/1, you'll get

{ "id": 1, "title": "json-server", "author": "typicode" }