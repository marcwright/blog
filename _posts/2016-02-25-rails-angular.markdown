---
layout: post
title: rails + angular
date:   2016-02-08 15:29:23 -0500
published: false
categories:
---

#Rails Angular Talk

##Why Rails?
A couple of key features of Rails. 

- Rails API is a great lightweight, poweru; server-side solution. You can add your front-end framework on top of it.

- It’s very structured and has a set of standards that the Rails community has agreed upon as best practice. So if another Rails dev is checking out your app, it’s easier to get up and running.
- allows you to scaffold apps very quickly so it's great for Startups, not too painful to roll out a prototype
  - AirBnb, Hulu, Twitter, Blue Apron, Plated, Daily Burn, GA website, Robinhood FAQ
- Auto-magical- can be good or bad
  - GOOD - rapid prototyping
  - BAD - Don’t always know what’s going on under the hood. As long as you play in their sandbox you’re ok   

#### Rails API
You can use rails for it's rails API only. It will scaffold out a project with no views. You can host Rails in one place and your front end in another.


## Rails App Instructions
- `rails new books -T --database=postgresql`
- `cd books`
- `bin/rake db:create`

## Add Angular using Bower
Now let's use bower to set up our front-end packages and dependencies. Bower was created by Twitter specifically to manage front-end assets

- add `gem 'bower-rails'` to the `Gemfile`
- create a `Bowerfile` in the root of your project and add the following:

~~~ruby
asset 'angular'
asset 'bootstrap-sass-official'
~~~

Then run `rake bower:install`. Bower installs dependencies in `vendor/assets/bower_components`. Rails asset pipeline dictates that you put your 3rd party code in vendor.

Since we're adding stuff outside of the normal flow of the asset pipeine we need to add it to `config/application.rb`.

```ruby
config.assets.paths << Rails.root.join("vendor","assets","bower_components")
config.assets.paths << Rails.root.join("vendor","assets","bower_components","bootstrap-sass-official","assets","fonts")
config.assets.precompile << %r(.*.(?:eot|svg|ttf|woff|woff2)$)
```
- add the paths to `application.js` and remove turbolinks

```ruby
require angular/angular
```
 - add to `application.css.scss` under all the comments and be sure to rename the file:

```scss
@import "bootstrap-sass-official/assets/stylesheets/bootstrap-sprockets";
@import "bootstrap-sass-official/assets/stylesheets/bootstrap";
```

- add `root 'home#index'` to `routes.rb`
- create `app/assets/controllers/home_controller.rb` and add the following:

```ruby
class HomeController < ApplicationController
  def index
  end
end
```

- create `app/assets/javascripts/app.js` and add the following:


~~~javascript
angular.module('receta',[
])
 .controller('NamesController', NamesController);

  function NamesController() {
    var vm = this;
    vm.marc = "Marc";
    vm.names = ["Marc", "Maren", "Diesel"]
  }
~~~


- create `app/assets/views/home/index.html.erb` and add:


~~~html
<div class="container-fluid" ng-app="receta">
  <div class="panel panel-success">
    <div class="panel-heading">
      <h1 ng-if="name">Hello, {{name}}</h1>
    </div>
    <div class="panel-body">
      <form class="form-inline">
        <div class="form-group">
          <input class="form-control" type="text" placeholder="Enter your name" autofocus ng-model="name">
        </div>
      </form>
    </div>
  </div>

  <div class="panel panel-success" ng-controller="NamesController as names">
    <div class="panel-heading">
      <h2>{{names.marc}}</h2>
      <ul ng-repeat="name in names.names">
        Hello, {{name}}
      </ul>
    </div>
  </div>
</div>
~~~


> You can also add `ng-app` to application.html.erb `body` 

####Scaffold Book model

- `rails g scaffold Books title author`
- `rake db:migrate`

`app.js`

```javascript
angular.module('receta',[
])
 .controller('NamesController', NamesController);

  function NamesController($http) {
    var vm = this;
    vm.marc = "Marc";
    vm.names = ["Marc", "Maren", "Diesel"];
    vm.books = getBooks().success(function(data){
      vm.books = data;
    });

    function getBooks(){
      return $http.get('http://localhost:3000/books.json');
    }
  }
```

####Scaffold Car model

- rails g model Car make model
- seed.rb:

```ruby
Car.create([
  {make: "Mazda", model: "CX-7"},
  {make: "VW", model: "Bug"}
  ])
```
- rake db:migrate db:seed

- using jbuilder `views/books/index.json.jbuilder`

```ruby
json.array!(@cars) do |car|
  json.extract! car, :id, :make, :model
end
```

-Cars Controller:

```ruby
class CarsController < ApplicationController
  
  def index
    @cars = Car.all
  end
end
```
-route.rb

```ruby
 get '/cars' => 'cars#index'
```

