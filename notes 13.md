# Rails as an API 

we can create a new rails app in API only mode using the --api flag  

```
rails new my_api --api
```

This will skip middleware and files that are used to support a full stack web app in rails. It will leave the pieces you need to build out a backend only API. 

What's sometimes confusing on the frontend is when you have 
```js
fetch('/posts')
  .then(res => res.json())
```

We need to parse the body of the response as a javascript data structure before we work with it. We get a string in JSON (JavaScript Object Notation) format back from the API, we need to turn it into an object or an array before we work with it in our frontend js.


The res.json() function sends a JSON response. This method sends a response (with the correct content-type) that is the parameter converted to a JSON string using the JSON.stringify() method.