Generate a scaffold so that we can quickly add a resource of Books
```sh 
rails g scaffold Book name:string category:string price:decimal isbn:integer
```

Run the migration to create the table - Books
```sh 
rake db:migrate
```

Add the following root route
```sh 
root 'books#index'
```

Add the form to the top of the index page
index.html.erb
```
<%= form_tag books_path, :method  => 'get' do %>
  <%= text_field_tag :search, params[:search] %>
  <%= submit_tag "search" %>
<% end %>
```

controller

Replace the index action with the following
``` 
def index
    @books = Book.all
end
```
Pour une recherche insensible à a la casse mettre un I devant Like
```
 def index
    @books = Book.where("name ILIKE ?","%#{params[:search]}%")
  end
  ```
  Transfer all the database logic to the model by creating a class method
  ```
  class Book < ActiveRecord::Base
  def self.search(search)
    if search
      where("name LIKE ?","%#{search}%")
    else
      all
    end
  end
end
```
Replace the index action with the following
```
def index
    @books = Book.search(params[:search])
  end
  ```
  
  ```sh 
  rails g model search keywords:string category:string min_price:decimal max_price:decimal isbn:integer
  ```
  
  Run the migration to create the table - Searches
  ```sh 
  rake db:migrate
  ```
  
  Generate the searches controller
  ```sh 
  rails g controller searches
  ```
  
  Add the the route - resources:searches
  ```sh 
  resources :searches
  ```
  
  Create the 3 actions we need - new / create / show
  ```
  class SearchesController < ApplicationController



  def new
    @search = Search.new
  end

  

  def create
    @search = Search.create(search_params)
    redirect_to search
  end



  def show
    @search = Search.find(params[:id])
  end
  
  

  private

  def search_params
    params.require(:search).permit(:keywords, :category, :min_price, :max_price, :isbn)
  end
end
```

Create a link to the advanced search
```
<%= link_to "Advanced Search", new_search_path %>
```

Create the form in the new template of the searches _controller
```
<h1>Advanced Search</h1>
 
<%= form_for @search do |s| %>
  <div class="field">
    <%= s.label :keywords %><br />
    <%= s.text_field :keywords %>
  </div>
  <div class="field">
    <%= s.label :category_id %><br />
    <%= s.select :category, options_for_select(@categories), :include_blank => true %>
  </div>
  <div class="field">
    <%= s.label :min_price, "Price Range" %><br />
    <%= s.text_field :min_price, size: 10 %> -
    <%= s.text_field :max_price, size: 10 %>
  </div>
  <div class="actions"><%= s.submit "Search" %></div>
  
<% end %>
```

Create a variable “@categories” in the new action so that you can fetch all the values of the categories column without repeating values
```
def new
   @search = Search.new
   @categories = Book.uniq.pluck(:category)
end
```
Pluck returns an Array of attribute values type-casted to match the plucked column names, if they can be deduced. 

Create a method called "search_books" and inside, the respective queries to the database
```
def search_books
   
   books = Book.all
   
   books = books.where("name ilike ?", "%#{keywords}%") if keywords.present?
   books = books.where("category like ?", category) if category.present?
   books = books.where("price >= ?", min_price) if min_price.present?
   books = books.where("price <= ?", max_price) if max_price.present?
   
   return books
end
```

Finally, create the show action to display the search results
```
<h1>Search Result</h1>
<% if @search.search_books.empty? %>
	<p>No Records Found</p>
<% else %>
	<% @search.search_books.each do |c| %>
		<div>	
			<h1><%= c.name %></h1>
			<p>Price: $<%= c.price %></p>
			<p>Category: <%= c.category %></p>
			<p>ISBN: <%= c.isbn %></p>
		</div>
	<% end %>
<% end %>
```
