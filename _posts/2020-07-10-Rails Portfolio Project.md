---
layout: post
title: Rails Portfolio Project Flatiron School (mod3 project)!
---

Rails contains alot of magic, it provides all kinds of generators that can make live easier. As part of our project we were told to stay away from the scaffold generator. The command <i>'rails new ordersystem'</i> already creates a big part of our project by creating the entire filestructure. What I learned is to be aware what version it will use during creating a new app. Two days in my project I discovered my project was using Rails 6.0.1. Our cohort lead has advised to switch to postgresql in stead of sqlite. During a live lecture he lead us through the setup and it was working without trouble on my side. Needless to say when re-initiating my project with Rails 5.2.3 I forgot to complete the <i>'rails new'</i> command with the database option to use postgress. when I noticed that and with the breakweek available I decided to start again with the correct Rails version and a Postgres database.

Different than in Sinatra, and maybe lacking in the magic, is that although having the option to configure the database automatically we have to run 'rails db:create' to create it.

The reason for the 'stay away from the scaffold generator' is that for a single model project it will generate literally everything for you to get things up and running very quick. Only the big question will be if the stuff generated is to your liking. Most likely you want things to be different and change things. Also the question will be, if you have to use generstors like scaffold do you really learn how rails works.

That still leaves us with a lot more but we will use the following generators:
- model
- resource
- controller
- migration

The model generator is the simplest one. When used it will generate a database migration file and a ruby model.<br>
<i>rails generate User name username email</i><br>
Will create:<br>
a migration in db/migrate/20200710934260_create_users.rb
{% highlight ruby %}
class CreateUsers < ActiveRecord::Migration[5.2]
  def change
    create_table :users do |t|
      t.string :name
      t.string :username
      t.string :email
      t.timestamps
    end
  end
end
{% endhighlight %}

the User model in app/models/user.rb
{% highlight ruby %}
class User < ApplicationRecord
end
{% endhighlight %}

In addition of this the resource generator will generate an entry in the /config/routes.db file to all the restful routes in the UserController that will also be generated.

The controller generator will generate the controller, the views folder, the helper for the controller and two files in the assets folder for javascript and stylesheet.
If you add an action to the end of the generator it will also create the view file for it.

The migration generator can be used to create a migration file either for the initial setup (create) of the table or to edit the content of the table (eg. rename column, add column)

All this creates the magic of Rails, more or less after setting up your application with this you only have to add some logic to your controller methods and you are up and running. It is a bit simplistic to state it like this as it still took me about a week to complete my project, but in big lines it is that simple.
