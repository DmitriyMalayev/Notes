# React Router
The idea with a react library like react router is that we integrate some JS logic that will read the URL and use the URL to decide what the user should see on that page. 
`Client Side Routing` 
Routing for a single page application 
A client side routing library like React Router is what allows us to have links that we can copy and share and provide users with the ability to access the same page in our app. 
We go from one URL to the next one wihin client side routing is not sending additional requests to the server automatically. It might indirectly cause that but it's not the same as the browser default. 


`Browser Default`
Sends a new request to a new url. 

import { HashRouter as Router, Route } from "react-router-dom";

// http://localhost:3000/#/about
// Works the same with the dev server. In production when you don't have a server compiling things. The HashRouter allows us to separate client side routes with a HashTag and then all of the requests that are sent with that HashTag are sent to the page in the rails app where your react app is served from rather than having a conditional statement to prevent errors. 
//HashRouter includes a HashTag before the client side route, the BrowserRouter does not. 
//If you have a web server with many things and want to distinguish between a request that should go to the server side path vs. a client side path. 


`Provider vs. Consumer`
Provider the one that does publishing 
Consumer is the one that takes in the published information 

`Redirect`
Allows us to push a URL into the history and allow react router render a new component 


// import { HashRouter as Router, Route } from "react-router-dom";

// http://localhost:3000/#/about
// Works the same with the dev server. In production when you don't have a server compiling things. The HashRouter allows us to separate client side routes with a HashTag and then all of the requests that are sent with that HashTag are sent to the page in the rails app where your react app is served from rather than having a conditional statement to prevent errors. 
//HashRouter includes a HashTag before the client side route, the BrowserRouter does not. 
//If you have a web server with many things and want to distinguish between a request that should go to the server side path vs. a client side path. 
