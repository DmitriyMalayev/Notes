**HTTP VERB**  GET 
**ROUTE** `/articles`
**ACTION** index action 
**USED FOR** Index Page To Display All Articles 

**HTTP VERB**  GET 
**ROUTE** `/articles/new`
**ACTION** New Action  
**USED FOR** Displays Create Article Form 

**HTTP VERB**  POST  
**ROUTE** `/articles` 
**ACTION** Create Action 
**USED FOR** Creates One Article 

**HTTP VERB**  GET   
**ROUTE** `/articles/:id` 
**ACTION** Show Action  
**USED FOR** Displays One Article Based on ID in the URL  



**HTTP VERB**  GET   
**ROUTE** `/articles:id/edit` 
**ACTION** Edit Action  
**USED FOR** Displays Edit Form Based on ID In The URL

**HTTP VERB**  PATCH 
**ROUTE** `/articles/:id` 
**ACTION** Update Action   
**USED FOR** Replaces An Existing Article Based On ID In The URL   

**HTTP VERB**  PUT
**ROUTE** `/articles/:id` 
**ACTION** Update Action  
**USED FOR** Replaces An Existing Article Based On ID In The URL

**HTTP VERB**  DELETE    
**ROUTE** `/articles/:id` 
**ACTION** Delete Action 
**USED FOR** Deletes One Article Based On ID In THe URL   


INDEX ACTION 

```ruby

#INDEX ACTION 

#This is an index action that allows the view to access all the articles in the DB through @articles 

get "/articles" do
    @articles = Article.all 
    erb :index 
end 

#NEW ACTION 

#Loads the form to create a new article.
get "/articles/new" do   
    erb :new 
end 

#CREATE ACTION 

#Responds to a POST request and creates a new article based on the params from the form and saves it to the DB.  
#Once the item is created, this action redirects to the show page. 

post "/articles" do 
    @article = Article.create(:title => params[:title], :content => params[:content])
    redirect to "/articles/#{@article.id}" 
end 

#SHOW ACTION

#In order to display a single article we need a show action. 
#This controller action responds to a GET request to he route "/articles/:id" 
#Because this route uses a Dynamic URL we can access the ID article in the view through the params hash.  

get "/articles/:id" do 
    @article = Article.find_by_id(params[:id])
    erb :show 
end 

#EDIT ACTION 


#Loads the edit form 
get "/articles/:id/edit" do 
    @article = Article.find_by_id(params[:id])
    erb :edit 
end 


#Handles the edit form submission. 
#This action responds to a PATCH request. 
#After saving this redirects to the article show page 

patch "/articles/:id" do 
    @article = Article.find_by_id(params[:id])
    @article.title = params[:title]
    @article.content = params[:content]
    @article.save 

    redirect to "/articles/#{@article.id}" 

end 



#PATCH Requests require us to include a hidden input field that will submit our form via patch

<form action="/articles/<%= @article.id %>" method="post">
    <input id="hidden" type="hidden" name = "_method" value="patch">
    <input type="text" name="title">
    <input type="text" name="content">
    <input type="submit" value="submit">
</form>
    

#The hidden input field uses Rack::MethodOverride  which is part of Sinatra Middleware
#In order to use middleware, and therefore use PATCH, PUT, or DELETE request you must tell your add to use the middleware by adding this line. 


use Rack::MethodOverride #This must be above run ApplicationController or any controller 
run ApplicationController 

#The middleware will run for every request sent by our app. 
#It will interpret any requests with name="_method" by translating the request to whatever is set by the value attribute
#In our example the POST REQUEST => PATCH REQUEST 
#PUT and DELETE are handled in a similar way


#DELETE ACTION 

#On the article show page, we have a form to delete it. 
#The form is submitted via a DELETE request to the route "/articles/:id" 
#This action finds the article in the DB based on the ID in the URL params and deletes it. 
#Then it redirects to "/articles"

delete "/articles/:id" do 
    @article = Article.find_by_id(params[:id])
    @article.delete 
    redirect to "/articles" 
end 

#This delete form needs the hidden input field 

<form action="/articles/<%= @article.id %> " method="post">
    <input id="hidden", type="hidden", name="_method", value="delete">
    <input type="submit" value="delete">
</form>


``` 





























