# Bcrypt-Guide

Uncomment Bcrypt gem in Gemfile line 27:
```ruby
gem 'bcrypt', '~>3.1.7'
```

##### Bundle the newly added Gem.
`bundle install`

##### Generate User Model
`rails g model User email password_digest admin:boolean`

##### Migrate
`rake db:create`
`rake db:migrate`

##### Generate corresponding User controller
`rails g controller users new`

##### add resources to routes.rb
```ruby
Rails.application.routes.draw do
  resources :users
```

##### Add to `models/user.rb`

Add has_secure_password to the Users model to enable the encryption from Bcrypt.
```ruby
class User < ActiveRecord::Base
  has_secure_password
end
```


##### Add to users-controller.rb
Add the actions to the Users controller as well as the private method of user_params.
The user_params method is used to filter the the fields that are expected to be given by User.
```ruby
def new
  @user = User.new
end

def create
  @user = User.new(user_params)
  if @user.save
    redirect_to root
  else
    redirect_to :new
  end
end

def show
end

def user_params
  params.require(:user).permit(:username, :email, :password, :password_confirmation, :admin)
end
```


##### Create the Signup form users/new.html.erb
Users signup form
```ruby
<h1> Sign Up </h1>

<%= form_for @user do |f| %>
  <p>
    <%= f.label :email %><br>
    <%= f.email_field :email %>
  </p>
  <p>
    <%= f.label :password %><br>
    <%= f.password_field :password %>
  </p>
  <p>
    <%= f.label :password_confirmation %><br>
    <%= f.password_field :password_confirmation %><br>
    <br>
  </p>
  <p>
    <%= f.label :admin %><br>
    <%= f.check_box %>
    <br>
    <%= f.submit %>
  </p>
  <% end %>
  </div>
```

A Session needs to be created for the login feature to actually work, otherwise you are just creating accounts.

##### Generate Sessions controller

`rails g controller sessions new`

##### Add Sessions resources to routes.rb
This enables your Sessions routes.
```ruby
Rails.application.routes.draw do
  root to: 'sessions/new'
  resources :users
  resource: :sessions
```

##### Sessions view sign-in.html.erb
This is the log in page for the user
```ruby
<h1> Log In </h1>
<%= form_tag sessions_path do %>
  <p>
    <%= label_tag :username %><br>
    <%= text_field_tag :username %>
  </p>
  <p>
    <%= label_tag :password %><br>
    <%= password_field_tag :password %>
  </p>
  <p>
    <%= submit_tag "Log In" %>
  </p>
<% end %>
```

###### Sessions controller
The create method confirms that the user information is correct, and if so will create a new session with that usetrs info.
```ruby
class SessionsController < ApplicationController
  def new
  end

  def create
    @user = User.find_by(email: params[:email]).try(:authenticate, params[:password])
    if @user
      session[:user_id] = @user.id
      redirect_to @user
    else
      redirect_to root_path
    end
  end
end
```
