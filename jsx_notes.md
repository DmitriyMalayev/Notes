# Dynamic React Components 
Building blocks of React Applications 
They describe a template of HTML and till in the variable data

`BlogContent`
Contains the content of the blog post 

`Comment`
Contains one user's comment 

`BlogPost`
The top level react component which is responding for rendering both BlogContent and Comment 

# Making Components Dynamic 

```js 
class BlogContent extends React.Component {
    render() {
        return(
            <div>
                {this.props.articleText}
            </div>
        )
    }
}

class Comment extends React.Component {
    render() {
        return (
            <div>
                {this.props.commentText}
            </div>
        )
    }
}

class BlogPost extends React.Component {
    render() {
        return(
            <div><>

            <div>
                <BlogContent articleText={"Dear Reader: Bjarne Stroustrup has the perfect lecutre oration"} />
                <Comment commentText={"I agree with this statement. -Angela Merkel"}/>
                <Comment commentText={"A universal truth. -Noam Chomsky"}/>
                <Comment commentText={"Truth is singular. It's 'versions' are mistruths. -Sonmi -451"}/>
            </div>
        )
    }
}
/*
We create a prop for BlogContent called article text and we assign it a value. 
The value is accessible from within BlogContent component as this.props.articleText 
Props can be any data type. 

We are passing information from a parent component to many child components. 
Specifically, we're doing this by creating a prop called commontText to pass each Comment component, which is then accessible in each instance of Comment as this.props.commentText. 


*/

# What's JSX? 
JSX allows us to write HTML like code in our JS files. 
JSX is a syntax extension of JS that creates a bond between HTML and JS. 

# Imperative vs. Declarative Programming 
`Imperative`
Write code that describes how something is done in detail 

`Declarative`
Write code explaining what you would like to do 

# What JSX Looks Like
JSX is the return value of the render() method 
Every component you use needs a render() method that returns some valid JSX. 
React components return JSX within their render() methods: 

```js 
class Tweet extends Component {
  currentTime = () => new Date().toString()

  render() {
    return (
      <div className="tweet">
        <img src="http://twitter.com/some-avatar.png" className="tweet__avatar" />
        <div className="tweet__body">
            <p>We are writing this tweet in JSX. Holy moly!</p>
            <p>{ Math.floor(Math.random()*100)} retweets </p>  
            <p>{ this.currentTime() }</p>
        </div>
      </div>
    );
  }
} 
``` 

# JSX can include JavaScript 
While writing our pseudo-HTML in JSX, we can also write vanilla javascript in line. We do this by wrapping the JS code in curly braces 
We call Math.floor() and Math.random() methods directly which will return a random number when the component is rendered. 
The custom function, currentTime() returns the string valud of the current date and time. 
We prepend it with `this` because it's a method within the Tweet class. 

# Arrow Functions 
We are implicitly binding the method to the Tweet class. 

# JSX Notes 
JSX cannot include all JS Statements
JSX is an extension of JS thaat wraps a lot of underlying function calls in a syntactically appealing style. 
This is why JSX is considered declarative. 

# Calling Class Methods In JSX
You can call class methods in jsx and within these methods you can include any javascript. 
A component must return one jsx element. 


```js 
class PlainDiv extends Component {
    render(){
        return <div> I am one line, so I do not need the parantheses </div>
    }
}

class Photo extends Component {
    render() {
        return (
            <figure> 
            <img className="image" src="httpsL//s3.amazonaws.com/ironboard-learn/sunglasses.gif" />
            <figcaption>Whoa</figcaption>
            </figure>
        )
    }
}

class Table extends Component {
    render(){
        return (
            <table> 
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                </tr>
                <tr>
                    <th>312213</th>
                    <th>Tim Berners-Lee</th>
                </tr>
            </table>
        )
    }
}

class ParentComponent extends Component {
    render(){
        return (
            <main>
                <PlainDiv />
                <Photo />
                <Table />
            </main>
        )
    }

}
```
Each one of those is a valid component, but all of them have one returned JSX element that contains everything else. Without an element that wraps the returned JSX element that contains everything else. Without an element that wraps the returned JSX in a component, we will get an error. There are some exceptions to this, such as React fragments, but most often, we will be using HTML-like JSX elements. 




# React Components 
Components let you split the UI into independent, reusable pieces, and think about each piece in isolation. 
Components modularize both functionality and presentation in our code. 
Each component contains a snippet of code that describes what it should render to the DOM. 

```js  
/*
A new class, Article is declared
The class extends React's component class, this provides us with build in methods and attributes. 
A render() method is defined and what it should return is explicitly provided. When you want to put this component on the DOM, here is what it should become. 
*/

class Article extends React.Component {
    render() {
        return(
            <div>
                Dear Render: Bjarne Stroustrup has the perfect lecture oration
            </div>
        )
    }
}


//Declaring a Comment class 
class Comment extends React.Component {
    render(){
        return(
            <div>
                Naturally, I agree with this article. 
            </div>
        )
    }
}

//Using the components in it's render method.  
//Every React application has some top level components. It's often called App. 

class App extends React.Component {
    render(){
        return 
        <div>
            <Article />
            <Comment />
        </div>
    }
}