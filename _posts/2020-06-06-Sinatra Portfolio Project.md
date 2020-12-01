---
layout: post
title: Sinatra Portfolio Project Flatiron School (mod2 project)!
---

# Sinatra Final Portfolio Project

Time went quick for me in mod 2. Started as a part timer and ended up being a full timer. This meant that the week following on my change I had to start thinking about a project as project week was starting right away.

I chose to make a **HamLogbook**, as Ham Radio is one of my hobbies and I thought would give plenty of room to apply the things we learned about Sinatra.

After learning the basics in mod 1 we took a dive into Sinatra. Although Sinatra takes away a lot of programming of what we have learned in mod 1, it brings some interesting features with the additional methods.
We learned about ActiveRecord, the Routes in Sinatra, the MVC structure of Modules, Views and Controllers, Sinatra and ActiveRecord and User Authentication. Almost too much in such a short time, but we managed to get a grasp of it, which we now can display in our Portfolio Project.

One of the interesting introduced gems is bcrypt, in combination with the ActiveRecord macro has_secure_password. The macro has_secure_password writes methods for us to set and authenticate against a Bcrypt password. The mechanism requires that we have a *(attribute)_digest* as attribute name for the desired password. Additionally it adds a requirement that a password is present the moment our User Object is created. It also checks to see that the password length is 72 bytes or less, and offers an optional password_confirmation attribute for password confirmation.

Bcrypt itself is a hashing mechanism that takes a digital fingerprint of a password. There is no way back, from the hash you cannot regenerate the password. Hackers however can create a database of hashed passwords using the bcrypt algorithm. If a database was compromised, hackers could do a reverse lookup of the hash to find the password. To make it more difficult for hackers a random chunk of data (salt) can be added to the password before it is hashed. This would make the database hackers would need to use to generate a possible salt and password hash extremely big in storage space and is most likely not done. So what is left for them in a plain dictionary attack including a possible salt. Herein lies the strength of bcrypt, as the algorithm is relatively slow compared with other algorithms like md5. It takes much longer to hash a password and thus you are slowing down the hacker guessing the password. 

Bcrypt works for us in the background providing us with a password= method that generates the encrypted password based on the given plaintext password and stores it in the password_digest.

Creating a new User instance without supplying a password:
```
 pry(main)> user = User.new(username: "john", password: "")
=> #<User:0x00007f80525718c0
 id: nil,
 username: "john",
 password_digest: nil,
 name: nil,
 email: nil>
 pry(main)> user.save
=> false
```

Althoug in the command line we are able to create a new instance of an User Object, we are not able to save it.
As User.create has the .save method build in, this will fail too.
```
pry(main)> user=User.create(username: "john", password: "")
=> #<User:0x00007f80525db248
 id: nil,
 username: "john",
 password_digest: nil,
 name: nil,
 email: nil>
 pry(main)> User.last
=> #<User:0x00007f8052618f80
 id: 4,
 username: "af5se",
 password_digest:
  "$2a$12$Qbg0kWJQSzdBcdM1wCc4Tefe4U4wAFEL/ADRdFOL1MUleIYXt/Mna",
 name: "John",
 email: "">
 pry(main)> 
```

As shown in the above section, the newly created User Object is not persisted to the database. In other words the .save method included in the .create method failed because a password was not present
If a password is supplied the create method will show the created object with an id value that is not nil. From this we see that the object has been persisted succesful into the database.

```
pry(main)> user=User.create(username: "alex", password: "1234")
=> #<User:0x00007f8052318f88
 id: 8,
 username: "alex",
 password_digest:
  "$2a$12$5GqNWDrtxQMnsmnmixU0N.WiflNh0BX/DMarA3JUlz0ya9bF5GUuS",
 name: nil,
 email: nil>
[13] pry(main)>
```

When we add a password the User Object will save succesfully and will be persisted to the database.

```
 pry(main)> user.password = '1234'
=> "1234"
 pry(main)> user.save
=> true
 pry(main)> User.last
=> #<User:0x00007f80554f54c0
 id: 7,
 username: "john",
 password_digest:
  "$2a$12$O4u602uMzjxy529e9VNX4.Oe73GbzNvjGRXVsXLagw6NySPldHVXa",
 name: nil,
 email: nil>
 pry(main)>
```

When we try to authenticate with the user with the correct password the User object will be returned.
```
pry(main)> user.authenticate('1234')
=> #<User:0x00007f80525db248
 id: 7,
 username: "john",
 password_digest:
  "$2a$12$O4u602uMzjxy529e9VNX4.Oe73GbzNvjGRXVsXLagw6NySPldHVXa",
 name: nil,
 email: nil>
 pry(main)>
```
With an wrong password we will see this.
```
pry(main)> user.authenticate('wrong password')
=> false
 pry(main)>
```


About my project, as mentioned above I decided to make HamLogbook. A place where you can store you contacts made with HAM radio.
I created three tables in my database, users, callsigns and contacts. With the logic that a user can have many callsigns and can have many contacts through callsigns. A callsign belongs to a user and a callsign can have many contacts. And at last the contacts belong to a callsign.

Normally a user would have enough with 1 call sign. A callsign is aplhabetically generated by the FCC and stays with the user as long as his license is valid (plus a 2 year grace period), independent of its class (Technician, General or Extra). At this moment new callsigns given out  consist of a combination of 2 letters, a regional number and 3 letters (mine in december 2017 was KG5WJQ). However it is possible to apply for a vanity callsign. Depending on the license class this could mean a much shorter call sign. That is something I did and obtained in 2018 K5GT which is also displayed in the webpages of HamLogbook. So in my case although my old callsign is no longer active I still have contacts that would be valid for a certificate logged with that callsign or combination of both.

This led in my HamLogbook to the decission that a user can have more than one callsign. This also created a challenge because the result of User.contacts would include all the contacts a user had made disregarding what callsign was used. So I had to find a way to pass the choice of callsign I wanted to see the contacts for. I solved this by storing the chosen callsign in the session so I could obtain it with session[:callsign]. This sesion[:callsign] is set in the CallsignController and set to nill either when the user logs out or when the user visits his homepage.

Because of my three tables, users, callsigns and contacts I also have three models with the same names, to complete the MVC (Model Views Controller) strcuture I also created views and controllers belonging to each.
The Model is where the object are defined, the views where the pages displaying the objects are rendered in erb files and controllers where the action takes places for the routing to the views and necessary data to be rendered. I created an extra SessionsController as that takes care of creating and logging in and out of the user and thus the session.

We are using CRUD, Create, Read, Update and Delete to manipulate our data we faced another challenge with our html form methods. Create and Read are not the problem because html forms have GET and POST. But for the UD part of our CRUD we need two more methods that are not implemented in the html forms. Lucky for use Rack came around the corner with the MethodOverride module which we can call for in our config.ru file with the line Rack::MethodOverride.
This module gives us the option to add an extra input line in our html form where we use `name="_``method"` and put the required post method in the value field `value="patch"` or `value="delete"`.

For example:
```
<form method="post" action="/contacts/<%=@contact.id%>">
            <input id="hidden" type="hidden" name="_method" value="patch">
```

One thing that I did find out is that the place in the config.ru file does matter. Make sure to place it above your controllers, I found it out the hard way and it did cost me some time. All the other routes of my controllers did work except for the patch and delete. I only saw the POST in my traces and never a PATCH.

All together it was fun and I learned a lot. Project came sooner than expected, but it totally put me on a different track now being fulltime. Next project I'm probably better prepared and I'm looking forward to it.
						