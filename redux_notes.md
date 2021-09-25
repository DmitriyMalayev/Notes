# JWT Auth in Redux and Rails

**This is a sample application and walks through _one_ possible auth implementation. It does not cover everything there is to know about auth and is intended as an introduction. Please do not blindly copy/paste the code here. Use this as a guide for setting up auth in a React/Redux application using JSON Web Tokens.**

- Second disclaimer: there are tradeoffs to every auth implementation. To secure our application further, we should set our tokens to expire and make sure our app is being served over [HTTPS](https://en.wikipedia.org/wiki/HTTPS). Furthermore, there are some [tradeoffs to storing JWTs in browser `localStorage`](https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage). [This StackOverflow post has a concise summary of the benefits/tradeoffs about where/how to store tokens client-side](https://stackoverflow.com/questions/35291573/csrf-protection-with-json-web-tokens/35347022#35347022).

---

## RAILS BCrypt, JWT 🔐

#### Building Our Server

- This section will walk through building a rails server. If you have questions about `Cors`, `ActiveModel::Serializer`, `Postgres`, namespacing and versioning our API, and/or general questions about Rails as an api only, refer [to this guide](https://github.com/learn-co-curriculum/mod3-project-week-setup-example).

- Let's create our app with `rails new backend_project_name --api --database=postgresql`

- We're going to need a few gems in our [Gemfile][gemfile] so let's go ahead and add them: `bundle add jwt && bundle add active_model_serializers && bundle add faker`––if you get a gem not found error, try running gem install on each of these, or manually add them to your [Gemfile][gemfile].

- Don't forget to uncomment `rack-cors` and `bcrypt` from your [Gemfile][gemfile].

- Call `bundle install`. Your [Gemfile][gemfile] should look something like this:

```ruby
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.5.1'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 5.2.1'
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 3.11'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
# gem 'jbuilder', '~> 2.5'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use ActiveModel has_secure_password
gem 'bcrypt', '~> 3.1.7'

# Use ActiveStorage variant
# gem 'mini_magick', '~> 4.8'

# Use Capistrano for deployment
# gem 'capistrano-rails', group: :development

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.1.0', require: false

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'


group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
end

group :development do
  gem 'listen', '>= 3.0.5', '< 3.2'
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end


# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

gem "jwt", "~> 2.1"

gem "active_model_serializers", "~> 0.10.7"

gem "faker", "~> 1.9"
```

- Don't forget to enable [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) in your app. Uncomment the following in [`config/initializers/cors.rb`][cors_rb]. Don't forget to change the origins from `example.com` to `*`
- Depending on the use-case and needs of our API, we might want to limit access to our app. For example, if our React frontend is deployed to `myDankReactApp.com`, we might want to limit access to that domain only. If certain endpoints are meant to be public, we can make those available but limit to `GET` requests, for example.

```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

- You can refer to the [rack-cors gem](https://github.com/cyu/rack-cors) for more information about this file.
- **Please don't forget to change these settings before deploying your app to the internet. _Please_**

---

#### Creating Users

- Run

  - `rails g model User username password_digest bio avatar`
  - `rails g controller api/v1/users`
  - `rails g serializer user` (if you want to [use a serializer](https://www.sitepoint.com/active-model-serializers-rails-and-json-oh-my/))
  - `rails db:create`
  - `rails db:migrate`

- Add `has_secure_password` to [`app/models/user.rb`][user_model]. Recall that `has_secure_password` comes from [`ActiveModel` and adds methods to set and authenticate against a BCrypt password](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password):

```ruby
class User < ApplicationRecord
  has_secure_password
end
```

- You might also want to add some [validations](https://guides.rubyonrails.org/active_record_validations.html) to your users:

```ruby
class User < ApplicationRecord
  has_secure_password
  validates :username, uniqueness: { case_sensitive: false }
end
```

---

#### Quick BCrypt Tangent

- Recall that `BCrypt` allows us to [salt](<https://en.wikipedia.org/wiki/Salt_(cryptography)>) users' plaintext passwords before running them through a [hashing function](https://en.wikipedia.org/wiki/Cryptographic_hash_function). A hashing function is, basically, a _one way_ function. Similar to putting something in a meat grinder: we cannot _feasibly_ reconstruct something that's been ground up by a meat grinder. We then store these passwords that have been 'digested' by `BCrypt` in our database. **[Never ever ever store your users' plaintext passwords in your database](https://blog.mozilla.org/webdev/2012/06/08/lets-talk-about-password-storage/). It's bad form and should be avoided at all costs.**

- Let's take a look at some of the functionality provided by `BCrypt`:

```ruby
# in rails console
> BCrypt::Password.create('P@ssw0rd')
 => "$2a$10$D0iXNNy/5r2YC5GC4ArGB.dNL6IpUzxH3WjCewb3FM8ciwsHBt0cq"
```

- `BCrypt::Password` [inherits from the Ruby `String` class](https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb#L23) and has its own [== instance method](https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb#L65) that allows us to run a plaintext password through `BCrypt` _using the same salt_ and compare it against an already digested password:

```ruby
# in rails console
> salted_pw = BCrypt::Password.create('P@ssw0rd')
  => "$2a$10$YQvJPemUzm8IdCCaHxiOOes6HMEHda/.Hl60cUoYb4X4fncgT8ubG"

> salted_pw.class
  => BCrypt::Password

> salted_pw == 'P@ssw0rd'
  => true
```

- `BCrypt` also provides a method that will take a stringified `password_digest` and turn it into an instance of `BCrypt::Password`, allowing us to call the over-written `==` method.

```ruby
# in rails console
> sample_digest = User.last.password_digest
  => "$2a$10$SJiIJnmQJ/A4z4fFG5EuE.aOoCjacFuQMVpVzQnhPSJKYLFCoqmWy"

> sample_digest.class
  => String

> sample_digest == 'P@ssword'
 => false

> bcrypt_sample_digest = BCrypt::Password.new(sample_digest)
  => "$2a$10$dw4sYcbLXc8XRX6YGc7ve.ot6LbYevMbSpFQZUaa8tm5NI8cxBPwa"

> bcrypt_sample_digest.class
  => BCrypt::Password

> bcrypt_sample_digest == 'P@ssw0rd'
  => true
```

![mind blown](https://media.giphy.com/media/26ufdipQqU2lhNA4g/giphy.gif)

- We have no way of storing instances of `BCrypt::Password` in our database. Instead, we're storing users' password digests **[as strings][schema]**. If we were to build our own `User#authenticate` method using `BCrypt`, it might look something like this:

```ruby
class User < ApplicationRecord
  attr_accessor :password

  def authenticate(plaintext_password)
    if BCrypt::Password.new(self.password_digest) == plaintext_password
      self
    else
      false
    end
  end
end
```

```ruby
# in rails console
> User.last.authenticate('not my password')
  => false

> User.last.authenticate('P@ssw0rd')
  => #<User id: 21, username: "sylviawoods", password_digest: "$2a$10$dw4sYcbLXc8XRX6YGc7ve.ot6LbYevMbSpFQZUaa8tm...", avatar: nil, created_at: "2018-08-31 02:11:15", updated_at: "2018-08-31 02:11:15", bio: "'Sylvia Woods was an American restaurateur who founded the sould food restaurant Sylvia's in Harlem on Lenox Avenue, New York City in 1962. She published two cookbooks and was an important figure in the community.">
```

- Instead of creating our own `User#authenticate` method, we can use [`ActiveModel#has_secure_password`](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password):

```ruby
class User < ApplicationRecord
  has_secure_password
end
```

![salt bae](https://media.giphy.com/media/l4Jz3a8jO92crUlWM/giphy.gif)

#### End of BCrypt Tangent

---

- Let's add a `create` method to our [`UsersController`][users_controller]:

```ruby
class Api::V1::UsersController < ApplicationController
  def create
    @user = User.create(user_params)
    if @user.valid?
      render json: { user: UserSerializer.new(@user) }, status: :created
    else
      render json: { error: 'failed to create user' }, status: :not_acceptable
    end
  end

  private
  def user_params
    params.require(:user).permit(:username, :password, :bio, :avatar)
  end
end
```

- We can use the [built in Rails HTTP status code symbols](https://gist.github.com/mlanett/a31c340b132ddefa9cca) when sending responses to the client; `status: :not_acceptable`, for instance. Need a refresher on [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)? Check out [httpstatusrappers.com/](http://httpstatusrappers.com/)

- And update our [`UserSerializer`][user_serializer]:

```ruby
class UserSerializer < ActiveModel::Serializer
  attributes :username, :avatar, :bio
end
```

---

- Next let's add the routes we'll need for our server. In [`config/routes.rb`][routes_rb]:

```ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :users, only: [:create]
      post '/login', to: 'auth#create'
      get '/profile', to: 'users#profile'
    end
  end
end
```

---

- Take some time to test this either in [Postman](https://www.getpostman.com/apps) or with JavaScript fetch:

```javascript
fetch('http://localhost:3000/api/v1/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json'
  },
  body: JSON.stringify({
    user: {
      username: "sylviawoods",
      password: "whatscooking",
      bio: "Sylvia Woods was an American restaurateur who founded the sould food restaurant Sylvia's in Harlem on Lenox Avenue, New York City in 1962. She published two cookbooks and was an important figure in the community.",
      avatar: "https://upload.wikimedia.org/wikipedia/commons/4/49/Syvia_of_Sylvia%27s_reaturant_N.Y.C_%28cropped%29.jpg"
    }
  })
})
  .then(r => r.json())
  .then(console.log)
```
**Note:** if you're using Postman and your formatting is set to "raw and JSON", remember to use double quotes ("") in both keys and values in the request.

---

## Make Sure You Can POST and Create a New User Before Proceeding

![intermission](https://media.giphy.com/media/pcPs6v6fhE7Ru/giphy.gif)

---

#### JSON Web Tokens (JWT)

- Token-based authentication is **stateless**. _We are not storing any information about a logged in user on the server_ (which also means we don't need a model or table for our user sessions). No stored information means our application can scale and add more machines as necessary without worrying about where a user is logged in. Instead, the client (browser) stores a token and sends that token along with every authenticated request. Instead of storing a plaintext username, or user_id, we can encode user data with JSON Web Tokens (JWT) and store that encoded token client-side.

---

#### JWT Auth Flow:

![](https://i.stack.imgur.com/f2ZhM.png)

- Here is the JWT authentication flow for logging in:
  1.  An already existing user requests access with their username and password
  2.  The app validates these credentials
  3.  The app gives a signed token to the client
  4.  The client stores the token and presents it with every request. This token is effectively the user's access pass––it proves to our server that they are who they claim to be.

* JWTs are composed of three strings separated by periods:

  ```
  aaaaaaaaaaaaaaa.bbbbbbbbbbbbbbbbbbbbb.ccccccccccccccccccc
  ```

  - The first part (aaaaaaaaaaaa) is the header

  - The second part (bbbbbbbbbbbb) is the payload - the good stuff, like who this person is, and their id in our database.

  - The third part (ccccccccccccc) is the signature. The signature is a hash of the header and the payload. It is hashed with a secret key, that we will provide (and should store in an environment variable using a gem like [Figaro](https://github.com/laserlemon/figaro#getting-started))

  - Head on over to [jwt.io](http://jwt.io/#debugger) and see for yourself:

  <img width="750" alt="JWTs" src="https://cloud.githubusercontent.com/assets/25366/9151601/2e3baf1a-3dbc-11e5-90f6-b22cda07a077.png">

---

### Encoding and Decoding JWTs

- Since we've already added [`gem jwt`](https://github.com/jwt/ruby-jwt) to our [gemfile][gemfile], let's explore some JWT methods by opening a `rails console`
  - `JWT.encode` takes up to three arguments: a payload to encode, an application secret of the user's choice, and an optional third that can be used to specify the hashing algorithm used. Typically, we don't need to show the third. This method returns a JWT as a string.
  - `JWT.decode` takes three arguments as well: a JWT as a string, an application secret, and––optionally––a hashing algorithm.

```ruby
#in rails console
>  payload = { beef: 'steak' }

> jwt = JWT.encode(payload, 'boeuf')
=> "eyJhbGciOiJIUzI1NiJ9.eyJiZWVmIjoic3RlYWsifQ._IBTHTLGX35ZJWTCcY30tLmwU9arwdpNVxtVU0NpAuI"

> decoded_hash = JWT.decode(jwt, 'boeuf')
=> [{"beef"=>"steak"}, {"alg"=>"HS256"}]

> data = decoded_hash[0]
=> {"beef"=>"steak"}
```

---

#### Building this functionality into our [`ApplicationController`][application_controller]:

- Given that many different controllers will need to [authenticate](https://en.wikipedia.org/wiki/Authentication) and [authorize](https://en.wikipedia.org/wiki/Authorization) users––[`AuthController`][auth_controller], [`UsersController`][users_controller], etc––it makes sense to lift the functionality of encoding/decoding tokens to our top level [`ApplicationController`][application_controller]. (Recall that **all** controllers inherit from [`ApplicationController`][application_controller])

```ruby
class ApplicationController < ActionController::API
  def encode_token(payload)
    # payload => { beef: 'steak' }
    JWT.encode(payload, 'my_s3cr3t')
    # jwt string: "eyJhbGciOiJIUzI1NiJ9.eyJiZWVmIjoic3RlYWsifQ._IBTHTLGX35ZJWTCcY30tLmwU9arwdpNVxtVU0NpAuI"
  end

  def decoded_token(token)
    # token => "eyJhbGciOiJIUzI1NiJ9.eyJiZWVmIjoic3RlYWsifQ._IBTHTLGX35ZJWTCcY30tLmwU9arwdpNVxtVU0NpAuI"

    JWT.decode(token, 'my_s3cr3t')[0]
    # JWT.decode => [{ "beef"=>"steak" }, { "alg"=>"HS256" }]
    # [0] gives us the payload { "beef"=>"steak" }
  end
end
```

---

- [According to the JWT Documentation](https://jwt.io/introduction/):
  Whenever the user wants to access a protected route or resource, the user agent (browser in our case) should send the JWT, typically in the Authorization header using the Bearer schema. The content of the header should look like the following:

  `Authorization: Bearer <token>`

---

- The corresponding `fetch` request might look like this:

```javascript
fetch('http://localhost:3000/api/v1/profile', {
  method: 'GET',
  headers: {
    Authorization: `Bearer <token>`
  }
})
```

---

- Knowing this, we can set up our server to anticipate a JWT sent along in request headers, _instead_ of passing the token directly to `ApplicationController#decoded_token`:

```ruby
class ApplicationController < ActionController::API
  def encode_token(payload)
    # payload => { beef: 'steak' }
    JWT.encode(payload, 'my_s3cr3t')
    # jwt string: "eyJhbGciOiJIUzI1NiJ9.eyJiZWVmIjoic3RlYWsifQ._IBTHTLGX35ZJWTCcY30tLmwU9arwdpNVxtVU0NpAuI"
  end

  def auth_header
    # { 'Authorization': 'Bearer <token>' }
    request.headers['Authorization']
  end

  def decoded_token
    if auth_header
      token = auth_header.split(' ')[1]
      # headers: { 'Authorization': 'Bearer <token>' }
      begin
        JWT.decode(token, 'my_s3cr3t', true, algorithm: 'HS256')
        # JWT.decode => [{ "beef"=>"steak" }, { "alg"=>"HS256" }]
      rescue JWT::DecodeError
        nil
      end
    end
  end
```

---

- A few things to note about the code above:
  - The [`Begin/Rescue` syntax](https://ruby-doc.org/core-2.2.0/doc/syntax/exceptions_rdoc.html) allows us to **rescue** out of an exception in Ruby. Let's see an example in a `rails console`. In the event our server receives and attempts to decode an **invalid token**:

```ruby
# in rails console
> invalid_token = "nnnnnnnooooooootttttt.vvvvvvaaaallliiiiidddddd.jjjjjjjwwwwwttttttt"

> JWT.decode(invalid_token, 'my_s3cr3t', true, algorithm: 'HS256')

Traceback (most recent call last):
        1: from (irb):6
JWT::DecodeError (Invalid segment encoding)
```

- In other words, if our server receives a bad token, this will raise an exception causing a [500 Internal Server Error](http://httpstatusrappers.com/500.html). We can account for this by **rescuing out of this exception**:

```ruby
# in rails console
> invalid_token = "nnnnnnnooooooootttttt.vvvvvvaaaallliiiiidddddd.jjjjjjjwwwwwttttttt"

> begin JWT.decode(invalid_token, 'my_s3cr3t', true, algorithm: 'HS256')
  rescue JWT::DecodeError
    nil
>  end
 => nil
```

- Instead of crashing our server, we simply return `nil` and keep trucking along.

![keep trucking](https://media.giphy.com/media/3xb5V0fbxHXck/giphy.gif)

---

- We can then complete our [`ApplicationController`][application_controller] by automatically obtaining the user whenever an authorization header is present:

```ruby
class ApplicationController < ActionController::API

  def encode_token(payload)
    # don't forget to hide your secret in an environment variable
    JWT.encode(payload, 'my_s3cr3t')
  end

  def auth_header
    request.headers['Authorization']
  end

  def decoded_token
    if auth_header
      token = auth_header.split(' ')[1]
      begin
        JWT.decode(token, 'my_s3cr3t', true, algorithm: 'HS256')
      rescue JWT::DecodeError
        nil
      end
    end
  end

  def current_user
    if decoded_token
      # decoded_token=> [{"user_id"=>2}, {"alg"=>"HS256"}]
      # or nil if we can't decode the token
      user_id = decoded_token[0]['user_id']
      @user = User.find_by(id: user_id)
    end
  end

  def logged_in?
    !!current_user
  end
end
```

- Recall that a Ruby object/instance is 'truthy': `!!user_instance #=> true` and nil is 'falsey': `!!nil #=> false`. Therefore `logged_in?` will just return a boolean depending on what our `current_user` method returns.

---

- Finally, let's lock down our application to prevent unauthorized access:

```ruby
class ApplicationController < ActionController::API
  before_action :authorized

  def encode_token(payload)
    # should store secret in env variable
    JWT.encode(payload, 'my_s3cr3t')
  end

  def auth_header
    # { Authorization: 'Bearer <token>' }
    request.headers['Authorization']
  end

  def decoded_token
    if auth_header
      token = auth_header.split(' ')[1]
      # header: { 'Authorization': 'Bearer <token>' }
      begin
        JWT.decode(token, 'my_s3cr3t', true, algorithm: 'HS256')
      rescue JWT::DecodeError
        nil
      end
    end
  end

  def current_user
    if decoded_token
      user_id = decoded_token[0]['user_id']
      @user = User.find_by(id: user_id)
    end
  end

  def logged_in?
    !!current_user
  end

  def authorized
    render json: { message: 'Please log in' }, status: :unauthorized unless logged_in?
  end
end
```

- A few things to note about the code above:
  - `before_action :authorized` will call the authorized method **before anything else happens in our app**. This will effectively lock down the entire application. Next we'll augment our [`UsersController`][users_controller] and build our [`AuthController`][auth_controller] to allow signup/login.

---

#### Updating the [UsersController][users_controller]

- Let's update the [UsersController][users_controller] so that it issues a token when users register for our app:

```ruby
class Api::V1::UsersController < ApplicationController
  skip_before_action :authorized, only: [:create]

  def create
    @user = User.create(user_params)
    if @user.valid?
      @token = encode_token(user_id: @user.id)
      render json: { user: UserSerializer.new(@user), jwt: @token }, status: :created
    else
      render json: { error: 'failed to create user' }, status: :not_acceptable
    end
  end

  private

  def user_params
    params.require(:user).permit(:username, :password, :bio, :avatar)
  end
end
```

- We need to make sure to skip the `before_action :authorized` coming from [ApplicationController][application_controller]

```ruby
class Api::V1::UsersController < ApplicationController
  skip_before_action :authorized, only: [:create]
end
```

- It wouldn't make sense to ask our users to be logged in before they create an account. This circular logic will make it **impossible** for users to authenticate into the app. How can a user create an account if our app asks them to be logged in or `authorized` to do so? Skipping the before action 'unlocks' this portion of our app.

![omg](https://media.giphy.com/media/l1J9Jzcs9OHXa310k/giphy.gif)

- Try creating a new user again with either [postman](https://www.getpostman.com/apps) or fetch and confirm that your server successfully issues a token on signup.

![sign me up gif](https://media.giphy.com/media/fSkMuhRxstDi5g5dwo/giphy.gif)

---

#### Implementing Login

- A token should be issued in two different controller actions: [`UsersController#create`][users_controller] and [`AuthController#create`][auth_controller]. Think about what these methods are responsible for––**a user signing up for our app for the first time** and **an already existing user logging back in**. In both cases, our server needs to issue a new token🥇.

- We'll need to create a new controller to handle login: `rails g controller api/v1/auth`. Next, let's add the following to this newly created [AuthController][auth_controller]:

```ruby
class Api::V1::AuthController < ApplicationController
  skip_before_action :authorized, only: [:create]

  def create
    @user = User.find_by(username: user_login_params[:username])
    #User#authenticate comes from BCrypt
    if @user && @user.authenticate(user_login_params[:password])
      # encode token comes from ApplicationController
      token = encode_token({ user_id: @user.id })
      render json: { user: UserSerializer.new(@user), jwt: token }, status: :accepted
    else
      render json: { message: 'Invalid username or password' }, status: :unauthorized
    end
  end

  private

  def user_login_params
    # params { user: {username: 'Chandler Bing', password: 'hi' } }
    params.require(:user).permit(:username, :password)
  end
end
```

- We can simply call our [`ApplicationController#encode_token`][application_controller] method, passing the found user's ID in a payload. The newly created JWT can then be passed back along with the user's data. **The user data can be stored in our application's state**, e.g., [React](https://reactjs.org/) or [Redux](https://redux.js.org/), while the token can be stored client-side.

- A few things to keep in mind about the code above:
  - `User.find_by({ name: 'Chandler Bing' })` will either return a user instance if that user can be found **OR** it will return `nil` if that user is not found.
  - In the event that the user is not found, `user = User.find_by(username: params[:username])` will evaluate to `nil`.
  - Can we call `.authenticate` on `nil`? NO!! `NoMethodError (undefined method 'authenticate' for nil:NilClass)`
  - Ruby, however, is **lazy**. If Ruby encounters `&&`, both statements in the expression must evaluate to true. If the statement on the left side evaluates to false, Ruby will **not even look at the statement on the right**. Let's see an example:

```ruby
# in irb or a rails console
> true && true
  => true

> true && false
  => false


> true && not_a_variable
  NameError (undefined local variable or method `not_a_variable` for main:Object)

> false && not_a_variable
  => false
```

- Let's take another look at our previous example:

```ruby
@user = User.find_by(username: params[:username])
if @user && @user.authenticate(params[:password])
end
```

- If `@user` is `nil`, which is falsey, **ruby will not even attempt to call `@user.authenticate`**. Without this catch, we'd get a `NoMethodError (undefined method 'authenticate' for nil:NilClass)`.

---

- Again, the client should be sending a JWT along with every authenticated request. Refer to this diagram from [scotch.io](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication):

![scotch.io article on token auth](https://cdn.scotch.io/scotchy-uploads/2014/11/tokens-new.png)

- A sample request might look like:

```javascript
fetch('http://localhost:3000/api/v1/profile', {
  method: 'GET',
  headers: {
    Authorization: `Bearer <token>`
  }
})
```

- So, let's update our [`UsersController`][users_controller] so that an authenticated user can access their profile information:

```ruby
class Api::V1::UsersController < ApplicationController
  skip_before_action :authorized, only: [:create]

  def profile
    render json: { user: UserSerializer.new(current_user) }, status: :accepted
  end

  def create
    @user = User.create(user_params)
    if @user.valid?
      @token = encode_token({ user_id: @user.id })
      render json: { user: UserSerializer.new(@user), jwt: @token }, status: :created
    else
      render json: { error: 'failed to create user' }, status: :not_acceptable
    end
  end

  private

  def user_params
    params.require(:user).permit(:username, :password, :bio, :avatar)
  end
end
```

- One final note about the snippet above: [`ApplicationController`][application_controller] calls `authorized` **before any other controller methods are called**. If authorization fails, our server will never call [`UsersController#profile`][users_controller] and will instead:

```ruby
render json: { message: 'Please log in' }, status: :unauthorized
```

---

## That's It For the Server!


---

### External Resources

- [HTTPS Wikipedia Article](https://en.wikipedia.org/wiki/HTTPS)
- [Storing JWTs in Cookies vs HTML5 localStorage](https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage)
- [StackOverflow Post on Cookies vs localStorage for Storing Tokens](https://stackoverflow.com/questions/35291573/csrf-protection-with-json-web-tokens/35347022#35347022)
- [Mod3 API Setup Guide](https://github.com/learn-co-curriculum/mod3-project-week-setup-example)
- [rack-cors gem](https://github.com/cyu/rack-cors)
- [MDN article on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Bcrypt gem](https://github.com/codahale/bcrypt-ruby)
- [Bcrypt::Password source code](https://github.com/codahale/bcrypt-ruby/blob/master/lib/bcrypt/password.rb#L23)
- [What is a Salt in Cryptography](<https://en.wikipedia.org/wiki/Salt_(cryptography)>)
- [What is a Cryptographic Hash Function](https://en.wikipedia.org/wiki/Cryptographic_hash_function)
- [ActiveModel has_secure_password docs](https://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password)
- [Mozilla Blog Post on Storing Passwords in a Database](https://blog.mozilla.org/webdev/2012/06/08/lets-talk-about-password-storage/)
- [ActiveModelSerializers gem](https://github.com/rails-api/active_model_serializers)
- [ActiveRecord Validations Documentation](https://guides.rubyonrails.org/active_record_validations.html)
- [SitePoint Article on ActiveModelSerializers in Rails](https://www.sitepoint.com/active-model-serializers-rails-and-json-oh-my/)
- [Postman App for making HTTP requests](https://www.getpostman.com/apps)
- [JWT Documentation](https://jwt.io/introduction/)
- [JWT Ruby Gem on GitHub](https://github.com/jwt/ruby-jwt)
- [JWT in Depth](https://blog.angular-university.io/angular-jwt/)
- [Scotch.io: The Ins and Outs of Token Based Authentication](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication)
- [Authentication](https://en.wikipedia.org/wiki/Authentication)
- [Authorization](https://en.wikipedia.org/wiki/Authorization)
- [Authentication vs Authorization](https://stackoverflow.com/questions/6556522/authentication-versus-authorization)
- [Figaro Gem for hiding secrets in your app](https://github.com/laserlemon/figaro#getting-started)
- [Ruby Begin Rescue Documentation](https://ruby-doc.org/core-2.2.0/doc/syntax/exceptions_rdoc.html)
- [HTTP Status Rappers](http://httpstatusrappers.com)
- [MDN Article on HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [Rails Status Code Symbols Cheat Sheet](https://gist.github.com/mlanett/a31c340b132ddefa9cca)
- [React Documentation](https://reactjs.org/)
- [Redux Documentation](https://redux.js.org/)

<!-- Markdown Variables -->

[gemfile]: /server/Gemfile
[schema]: /server/db/schema.rb
[application_controller]: /server/app/controllers/application_controller.rb
[auth_controller]: /server/app/controllers/api/v1/auth_controller.rb
[users_controller]: /server/app/controllers/api/v1/users_controller.rb
[user_model]: /server/app/models/user.rb
[user_serializer]: /server/app/serializers/user_serializer.rb
[cors_rb]: /server/config/initializers/cors.rb
[routes_rb]: /server/config/routes.rb 


# Why Redux?

## Objectives

In this lesson, we will learn about the Redux architecture for building web
applications. By the end of the lesson you will be able to:

* Understand how __Redux__ encourages a single source of truth.
* Understand how actions fit into the Redux flow.

## Benefits of moving to Redux

#### Single Source Of Truth

As our __React__ applications become larger, our state becomes more spread out
between different components.  At a certain point, the component tree becomes a
web of props and state that can obscure our view of how components are handling
and sharing data with each other.

There are ways to get around this, like storing all of our state in one high
level container component, but this can ultimately _increase_ the complexity of
your props.

__Redux__ offers a different solution. It encourages storing all of the
necessary data in our application in a JavaScript object _separate_ from our
components.  It looks like this:

```javascript
state = {
  user: {
    name: 'bob',
    hometown: 'philly'
  },
  interests: [
    {
      name: 'pokemon',
      type: 'game'
    },
    {
      name: 'game of thrones',
      type:'tv show'
    }
  ]
}
```

Similar to component state, all our data is held in an object. The difference
here is that, since Redux state is separate from the component tree, we can grab
_any_ part of this data for _any_ component that needs it, just by connecting
the component!

#### Accessing Our State

To make this state available for components to connect to, we provide access by
wrapping the component tree, similar to `Router`. This gives us access to Redux
functions that allow us to grab state and map it the props being given to a
component. Components can then read these props like normal, as though they were
receiving them from a parent component.

Consequently, complex interaction between components is made easier. Take for
example sibling components (rendered side by side in a parent) and cousin
components (the children of sibling components). If siblings are both displaying
or manipulating the same bit of shared data, without Redux, that data needs to
be stored in their parent component's state. If _cousins_ are sharing data, the
data needs to be stored in the _grandparent_ component, the closest shared
'ancestor' component.

In Redux all these interactions are structured the same way. Every component we
allow can get and update state data regardless of the position of components in
a tree.

#### Updating Our state.

So we hold all of our data in one place and with some configuration, we can read
it as props in regular React components. When we want to update that data, we
must send an action, which is a set of strict instructions _we create_ that
__Redux__ will use for how to update it.  

```javascript
action = {
  type: 'ADD_INTEREST',
  newInterest: {
    name: 'hockey',
    type: 'sport'
  }
}
```

Here, we can imagine that after a user fills out a form and clicks submit, we
will create an action that tells __Redux__ how to incorporate the update into
the state.  Any time we update the state in __Redux__, we must create an action
first. This action is just a plain old JavaScript object.

These actions are also made available to components. Any component we connect
will be able to modify the state using an action we've defined.

Following a specific design pattern that we'll explore through the upcoming
lessons, we can use these actions to maintain and update our Redux state. These
state changes trigger React's component lifecycle, and thus, the cycle of data
is made.

## Summing Up

__Redux__ places all of our data in one place -- the state.  This state is just
a plain JavaScript object. In fact, all the pieces of Redux are plain old
JavaScript. It is the pattern, the way the information flows that makes Redux
awesome.

To change our application state, we need create an action that holds information
for how to update that state. The action, combined with the previous state,
produces an updated state.

## Resources

* [Redux Justification - Dan Abramov](https://www.youtube.com/watch?v=xsSnOQynTHs)
* [Looking back at Redux - Dan Abramov](https://www.youtube.com/watch?v=uvAXVMwHJXU)

<p class='util--hide'>View <a href='https://learn.co/lessons/why-redux'>Why Redux</a> on Learn.co and start learning to code for free.</p>




# Using Pure Functions to Update Application State

## Objectives:

In this lesson, you will learn the following:

* Learn how pure functions update our application state.
* Learn the fundamentals of the redux flow.
* Learn how to adhere to the constraints of a pure function.

## Review

So far we know that all of our state is in a JavaScript object, and that our
actions are in another JavaScript object called an action. Somehow the action
updates our state.

Let's take a look at an example:

```js
let state = {count: 0}
let action = {type: 'INCREASE_COUNT'}
```

Somehow I want to send this action to the state so that at the end our state is
updated to look like the following: `state -> {count: 1}`.

But how??

## Functions to the Rescue

This seems easy enough. Why not just write a function that takes in our previous
state, takes in our action, and depending on that action produces a new state.
Here's what it could look like:

```js
function changeState(state, action) {
  if (action.type === 'INCREASE_COUNT') {
    return {count: state.count + 1 }
  }
}
```

That's pretty straightforward code. If the action's type property is the String
`'INCREASE_COUNT'` then go ahead and increment and return the new state.

The important piece of information we looked at to determine how to change the
state was `action.type`. Actions always need a `type` property so the function
knows what to do. If you can imagine a whole bunch of different actions that
change the state in different ways, `'DECREASE_COUNT'`,
`'INCREASE_COUNT_BY_TEN'` and so on, it shouldn't be hard to see how that code
could become very messy with a bunch of `if`s and `else if`s. Instead, it is
customary to use a `switch case` statement.

```js
function changeState(state, action){
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
  }
}
```

This makes it very explicit and clear that `action.type` is the information we
are switching on to make our decision on how to change the state.

We'll talk about this more in-depth later, but it is important that when we
change the state we never return `null` or `undefined`. We'll cover this by
adding a `default` case to our function.

```js
function changeState(state, action){
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    default:
      return state;
  }
}
```

This way, no matter what, when accessing the Redux state we'll always get some
form of the state back.

```js
let state = {count: 0}
let action = {type: 'INCREASE_COUNT'}

changeState(state, action)
// => {count: 1}
```

Ok, so if you copy and paste that code into a JavaScript console, you'll see
that the function works just as we'd expect. The state and action are passed to
our `changeState function`, which hits the `'INCREASE_COUNT'` case statement.
Then it takes the state's count of zero, adds one to it, and returns a new
object `{count: 1}`.

Now let's have this function respond to another action, decrease count. Give it
a shot, the answer is below.


```js
function changeState(state, action){	  
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    case 'DECREASE_COUNT':
      return {count: state.count - 1}
    default:
      return state;
  }
}

let state = {count: 0}

changeState(state, {type: 'INCREASE_COUNT'})
// => {count: 1}

changeState(state, {type: 'DECREASE_COUNT'})
// => {count: -1}
```

Ok! That my friends, is the crux of redux. To summarize:

```
Action -> Function -> Updated State
```

And let's give this function a name. Because it is combining two pieces of
information, our current state and an action, reducing this combination into one
value, we'll say that it _reduces_ the two into one updated state. For this
reason, we call this function a reducer:

```
Action -> Reducer -> Updated State
```

As you learn more about redux, things may become more complex. Just remember
that at the core of redux is always this flow. An action gets sent to a reducer
which then updates the state of the application.

# TWIST

You may notice a problem. While we can call the changeState reducer to increase
the count from zero to one, if we call change state again we keep returning a
count of one. In other words, we are not persisting this change of state. We'll
tackle how this works in an upcoming section.

## Reducers are pure functions

```js
function reducer(state, action){	  
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    case 'DECREASE_COUNT':
      return {count: state.count - 1}
    default:
      return state;
  }
}
```

An important thing to note about reducers is that they are pure functions. Let's
remember the characteristics of pure functions:

1. Pure functions are only determined by their input values

2. Pure Functions have no side effects. By this we mean pure functions do not have
any effect outside of the function. They only return a value.

> Note: The reason we like pure functions so much is because if a function has
no effect outside of the object, and if the function always returns the same
value given a specific input, this means that our functions become really
predictable. In addition, the lack of side effects means that the functions are
also contained, and can be used safely without affecting the rest of your
application.

Let's take these two characteristics of pure functions in turn, and ensure that
we are adhering to them here.

Ok, so the first characteristic of pure functions means that given the same
input of the function, I will always receive the same output from that function.
That seems to hold, given a specific state object like `{count: 2}` and an
action object like `{type: 'DECREASE_COUNT'}` will I always get back the same
value? Yes. Given those two arguments, the output will always be `{count: 1}`.

As for the 'no side effects' characteristic, there's something pretty subtle
going on in our reducer. The object returned is not the same object that is
passed as an argument to the function, but rather a new object that is
constructed each time our reducer is called. Do you see why? Take a close look
at the line that says `return {count: state.count + 1}`. This line is
constructing a new JavaScript object and setting its count attribute to equal
the previous state's count plus one. So we adhere to the constraints of a pure
function by not changing any value that is defined outside of the function.

## Summary

1. We hold our application's state in one plain old JavaScript object, and we
update that state by passing both an action and the old state to our reducer.
Our reducer returns to us our new state.
2. So to change our state we (1) create an action (an **action** is just a plain 
object with a type key); and (2) and pass the action as an argument when we call 
the **reducer** (which is just a function with a switch/case statement). This 
produces a new state.
3. Our reducer is a pure function which means that given the same arguments of
state and action, it will always produce the same new state. Also it means that
our reducer never updates the previous state, but rather creates a new state
object.

<p class='util--hide'>View <a href='https://learn.co/lessons/redux-flow'>Redux Flow</a> on Learn.co and start learning to code for free.</p>






# Reducers Lab

![reduce](https://media.giphy.com/media/3o7TKwxYkeW0ZvTqsU/giphy.gif)

## Objectives

1. Write a reducer.
2. The reducer should be a pure function.
3. Write a reducer that takes an action(payload).

## Overview

Each year, around the holidays especially, we forget who we need to buy presents
for. Let's write a function that will help us manage our gift recipient list. We
should be able to add a person we need to buy a present for and remove people we
no longer like (or who give us socks every year!).

In this lab, we will focus on writing two reducers to help us handle this task.
Since we're drilling down into one concept, React is not set up in this lab, nor
is there an `index.html` file. Follow the instructions below and implement your
code in `src/reducers/manageFriends.js` and `src/reducers/managePresents.js`.

## Instructions

1. In `managePresents.js`, write a function called `managePresents()` that takes
   in the previous state and an action as its argument. Set an default value for
   the state argument - an object with a key, `numberOfPresents`, assigned to
   `0`.

   Actions passed into this reducer will only have a _type_ attribute, so they
   would look something like this:

      ```js
        action = {
          type: "INCREASE"
        }
      ```

   If the reducer receives a `type` set to `"INCREASE"`, return a new state
   where the value of `numberOfPresents` is increased by one. Use the tests to
   guide you as you build out this reducer.

2. In `manageFriends.js`, write a function called `manageFriends` that takes in
   the previous state and an action as its argument.  Here, the initial state
   should be an object with a key, `friends`, set to an empty array.

   This time, the reducer should be able to handle two actions, `"ADD_FRIEND"`
   and `"REMOVE_FRIEND"`. When adding a friend, the action will include a
   `friend` key assigned to an object with `name`, `hometown`, and `id` keys.

    ```js
      action = {
        type: "ADD_FRIEND",
        friend: {
          name: "Chrome Boi"
          homewtown: "NYC",
          id: 1
        }
      }
    ```

   When our reducer receives `"ADD_FRIEND"`, it should return a new state with
   this friend object added to the `friends` array.

   When removing a friend, instead of an object, the action will include an `id` key
   with an integer. Find the friend with the matching `id` and remove them. Thought of
   in another way, the reducer is really returning a new state with an array of `friends`
   that includes everyone _except_ the removed friend.

    ```js
      action = {
        type: "REMOVE_FRIEND",
        id: 1
      }
    ```

Both reducers should be pure functions.  This means that the functions cannot
change any object defined outside of the functions.  It also means that given an
input, the reducers will always return the same output.

## Don't Mutate State

As the Redux documentation notes:

> Since one of the core tenets of Redux is to never mutate state, you'll often
find yourself using `Object.assign()` to create copies of objects with new or
updated values.

If you remember, `Object.assign()` is a function that takes any number of
arguments. It works by copying over from left to right the properties in each
object passed as an argument.  Let's go over an example:

```js
let dog = {id: 1, name: 'scooby', color: 'brown', age: 4};
// if scooby had a birthday, we could write:
let olderDog = Object.assign({}, dog, {age: dog.age + 1})
```

Translating this to English would be something like, "Start with a new empty
object, copy over everything from the original `dog`, then overwrite the `age`
property with a new value."

## A Note on the Object Spread Operator

![future](https://media.giphy.com/media/l0CRCmMBYQbL7dCmI/giphy.gif)

While effective, using `Object.assign()` can quickly make simple reducers
difficult to read given its rather verbose syntax.

An alternative approach is to use the object spread syntax, which lets you use
the spread (...) operator to copy enumerable properties from one object to
another in a more succinct way

```javascript
let dog = {id: 1, name: 'scooby', color: 'brown', age: 4};

let olderDog = {...dog, age: dog.age + 1}
```

This would translate to the same English, "Return a new object that contains all
the key-value pairs from `dog` copied over with the `age` key overwritten with a
new value".

## Resources

- [Mozilla Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
- [Redux Docs: Object Spread Operator](http://redux.js.org/docs/recipes/UsingObjectSpreadOperator.html)
- [Mozilla findIndex](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)
- [Redux Documentation](http://redux.js.org/docs/basics/Reducers.html)


# Persisting State and Displaying State

## Objectives

In this code-along, you will learn the following:

* How to persist changes to state.
* How to display changes to the state in the view.

An HTML page, `index.html`, is provided and already linked to `js/reducer.js`.
Open `index.html` in browser to access the functions in `reducer.js` and follow
along.

## Building our Counter Application

Previously, we've had a reducer that updates state. In the example we used, we
defined a `switch` statement with one `case` and a default:

```javascript
function changeState(state, action){
  switch (action.type) {
    case 'INCREASE_COUNT':
      return {count: state.count + 1}
    default:
      return state;
  }
}

let state = {count: 0}
let action = {type: 'INCREASE_COUNT'}

changeState(state, action)
// => {count: 1}
```

## Persisting State

We currently have a problem.  Let's call `changeState` multiple times:

```javascript
changeState(state, {type: 'INCREASE_COUNT'})
  // => {count: 1}
changeState(state, {type: 'INCREASE_COUNT'})
  // => {count: 1}
changeState(state, {type: 'INCREASE_COUNT'})
  // => {count: 1}
```

See that? Our state never increases beyond one.  It starts off as zero, and
while the `changeState` function returns one more than that, if we look at
state, it still returns `{count: 0}`.  Now, fixing this in the console isn't so
bad.  We just write

```javascript
state = changeState(state, {type: 'INCREASE_COUNT'})
state
  => {count: 1}
state = changeState(state, {type: 'INCREASE_COUNT'})
  => {count: 2}
```

Here, we're reassigning state to the return value of our reducer. This way, the
second time `changeState` is called, it is using the updated state in its
arguments.

Ok.  So let's encapsulate this procedure in a function so that we can just call
that method and it will persist our changes.  We'll name that function
`dispatch`.  

```javascript
let state = {count: 0};

function changeState(state, action){
    switch (action.type) {
      case 'INCREASE_COUNT':
        return {count: state.count + 1}
      default:
        return state;
    }
  }

function dispatch(action){
  state = changeState(state, action)
  return state
}

dispatch({type: 'INCREASE_COUNT'})
  // => {count: 1}
dispatch({type: 'INCREASE_COUNT'})
  // => {count: 2}
dispatch({type: 'INCREASE_COUNT'})
  // => {count: 3}
```

Ok, it's fixed!!  Previously our state was stuck at one, but now we just call
our `dispatch` function, and pass through our action.  Let's walk through it.

We declare our state to equal an object `{count: 0}`.  Then, we define our
reducer and our new function `dispatch`.  At the bottom, we call the `dispatch`
function and pass through our action, `{type: 'INCREASE_COUNT'}`.  When we call
`dispatch`, this calls our `changeState` reducer, and passes the action object
to the reducer.  When called, the `changeState` reducer also takes in `state`,
which has been declared up above.  

`state` is assigned the return value of `changeState`. Since the
`INCREASE_COUNT` type was used, the returned value of `changeState` contains a
`count` equal to the previous state's count plus one.

Thus, our state is updated. Each time `dispatch` is called, the current version of
`state` is passed into `changeState`, and then `state` is assigned a new value based
on what `changeState` returns.

## Rendering Our State

Ignoring React for a bit, how would we display something like this on a page?
And how would we make sure that our HTML updates every time we change our state?
Well, to render this on the page we can write a render function that changes our
HTML:

```javascript
function render(){
  document.body.textContent = state.count
}
```

Now if we want to call our render function, we'll see our state on the page:

```javascript
let state = {count: 0};

function changeState(state, action){
    switch (action.type) {
      case 'INCREASE_COUNT':
        return {count: state.count + 1}
      default:
        return state;
    }
  }

function dispatch(action){
  state = changeState(state, action)
  return state
}

function render(){
  document.body.textContent = state.count
}

// call the render function
render()
```

And presto! Our number is displayed on the page.  However, we want to call
`render` every time our state changes.  And it's safe to say our state will not
change unless we call the `dispatch` function.  So we can ensure that the
`render` function runs every time that our `dispatch` function is called by
changing our `dispatch` function to the following.

```javascript
function render(){
  document.body.textContent = state.count
}

function dispatch(action){
  state = changeState(state, action)
  render()
}

dispatch({type: 'INCREASE_COUNT'})
dispatch({type: 'INCREASE_COUNT'})
```

## Summary

Just to show everything together finally:

```js
let state = {count: 0};

function changeState(state, action){
    switch (action.type) {
      case 'INCREASE_COUNT':
        return {count: state.count + 1}
      default:
        return state;
    }
  }

function render(){
    document.body.textContent = state.count
}

function dispatch(action){
  state = changeState(state, action)
  render()
}

render()
```

If you copy the code above into `js/reducer.js` and open `index.html` in 
your browser, in the console, you can confirm everything is working by running `dispatch({type: 'INCREASE_COUNT'})` and watch as the displayed
count increases!

With just this set of functions, we could actually apply our own Redux pattern
to a regular ol' JavaScript and HTML webpage!

## Conclusion

In this lesson, we've introduced a new function called the `dispatch` function.
Our `dispatch` function solved two problems for us.

First, it persisted changes to our state.  This is because we called the
`dispatch` function, the `dispatch` function called our reducer, and then we
took the return value from the reducer and assigned it to be our new state.

Second, it ensured that each time our state updates, our HTML updates to reflect
these changes.  It does this by simply calling the `render` function.  Each time
we call `dispatch` it's as if we are then calling render.  Don't worry about
re-rendering too much.  Remember that when we integrate with React, React will
only be virtually re-rendering the DOM, and then updating the DOM with the
smallest number of changes to ensure a performant application.

<p class='util--hide'>View <a href='https://learn.co/lessons/redux-dispatch'>Redux Dispatch</a> on Learn.co and start learning to code for free.</p>




# Reducers Lab

![reduce](https://media.giphy.com/media/3o7TKwxYkeW0ZvTqsU/giphy.gif)

## Objectives

1. Write a reducer.
2. The reducer should be a pure function.
3. Write a reducer that takes an action(payload).

## Overview

Each year, around the holidays especially, we forget who we need to buy presents
for. Let's write a function that will help us manage our gift recipient list. We
should be able to add a person we need to buy a present for and remove people we
no longer like (or who give us socks every year!).

In this lab, we will focus on writing two reducers to help us handle this task.
Since we're drilling down into one concept, React is not set up in this lab, nor
is there an `index.html` file. Follow the instructions below and implement your
code in `src/reducers/manageFriends.js` and `src/reducers/managePresents.js`.

## Instructions

1. In `managePresents.js`, write a function called `managePresents()` that takes
   in the previous state and an action as its argument. Set an default value for
   the state argument - an object with a key, `numberOfPresents`, assigned to
   `0`.

   Actions passed into this reducer will only have a _type_ attribute, so they
   would look something like this:

      ```js
        action = {
          type: "INCREASE"
        }
      ```

   If the reducer receives a `type` set to `"INCREASE"`, return a new state
   where the value of `numberOfPresents` is increased by one. Use the tests to
   guide you as you build out this reducer.

2. In `manageFriends.js`, write a function called `manageFriends` that takes in
   the previous state and an action as its argument.  Here, the initial state
   should be an object with a key, `friends`, set to an empty array.

   This time, the reducer should be able to handle two actions, `"ADD_FRIEND"`
   and `"REMOVE_FRIEND"`. When adding a friend, the action will include a
   `friend` key assigned to an object with `name`, `hometown`, and `id` keys.

    ```js
      action = {
        type: "ADD_FRIEND",
        friend: {
          name: "Chrome Boi"
          homewtown: "NYC",
          id: 1
        }
      }
    ```

   When our reducer receives `"ADD_FRIEND"`, it should return a new state with
   this friend object added to the `friends` array.

   When removing a friend, instead of an object, the action will include an `id` key
   with an integer. Find the friend with the matching `id` and remove them. Thought of
   in another way, the reducer is really returning a new state with an array of `friends`
   that includes everyone _except_ the removed friend.

    ```js
      action = {
        type: "REMOVE_FRIEND",
        id: 1
      }
    ```

Both reducers should be pure functions.  This means that the functions cannot
change any object defined outside of the functions.  It also means that given an
input, the reducers will always return the same output.

## Don't Mutate State

As the Redux documentation notes:

> Since one of the core tenets of Redux is to never mutate state, you'll often
find yourself using `Object.assign()` to create copies of objects with new or
updated values.

If you remember, `Object.assign()` is a function that takes any number of
arguments. It works by copying over from left to right the properties in each
object passed as an argument.  Let's go over an example:

```js
let dog = {id: 1, name: 'scooby', color: 'brown', age: 4};
// if scooby had a birthday, we could write:
let olderDog = Object.assign({}, dog, {age: dog.age + 1})
```

Translating this to English would be something like, "Start with a new empty
object, copy over everything from the original `dog`, then overwrite the `age`
property with a new value."

## A Note on the Object Spread Operator

![future](https://media.giphy.com/media/l0CRCmMBYQbL7dCmI/giphy.gif)

While effective, using `Object.assign()` can quickly make simple reducers
difficult to read given its rather verbose syntax.

An alternative approach is to use the object spread syntax, which lets you use
the spread (...) operator to copy enumerable properties from one object to
another in a more succinct way

```javascript
let dog = {id: 1, name: 'scooby', color: 'brown', age: 4};

let olderDog = {...dog, age: dog.age + 1}
```

This would translate to the same English, "Return a new object that contains all
the key-value pairs from `dog` copied over with the `age` key overwritten with a
new value".

## Resources

- [Mozilla Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
- [Redux Docs: Object Spread Operator](http://redux.js.org/docs/recipes/UsingObjectSpreadOperator.html)
- [Mozilla findIndex](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)
- [Redux Documentation](http://redux.js.org/docs/basics/Reducers.html)

