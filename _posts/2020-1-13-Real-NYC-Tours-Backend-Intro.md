---
layout: post
title: Real NYC Tours Backend Intro
---

**Real NYC Tours** is the name of my new React Native side project. The idea is the app has checklists of fun places to visit in NYC, and these are places that tourist might not otherwise hear about. There might be a _Secret Brooklyn_ checklist of destinations, and there might be another _Best Baby Places in the Upper West Side_ checklist with fun spots for familes.

{% include image.html url="/images/IMG_0596.jpeg" description="Me and Teddy know where babies like on the UWS" %}

The client is built with React Native, and you can see the current state of it [here](https://github.com/jeffthemaximum/nycTourApp). The backend is a Ruby on Rails API that's [here](https://github.com/jeffthemaximum/nyc_tour_backend).

I've been trying to invite some of my friends to work on this project with me, but they're not all familiar with Ruby or Rails. I'm hoping this post might help familiarize them with how my Rails backend works.

Two pieces of information regarding the following documention:
 1. It is meant for experienced developers who are newer to Rails; It's not intended for newer programmers in general.
 2. It is meant to provide some information about how to use this backend - not to provide every details about how ruby and rails work internally.

## How the backend works

### high-level overview

Rails is a Model, View, Controller (MVC) framework. This means you have *model* files that handle interactions with the database, *controller* files that are the code that executes for a particular endpoint, and *view* files that are the HTML for a webpage. [NYC Tours](https://github.com/jeffthemaximum/nyc_tour_backend) uses Rails in `API` mode, where the main difference is the views are not HTML but JSON.

Rails gets criticized for having a lot of "magic" - or code and functionality that's sometimes hidden from the developer. This is valid criticism, and it sometimes makes things confusing for Rails developers.

The main Rails "magic" that [NYC Tours](https://github.com/jeffthemaximum/nyc_tour_backend) uses is the lifecycle methods available in the *model* and *controller* files. Just like React has lifecycle methods that fire before and after `render` is executed (such as `componentWillMount` and `componentReceivedProps`), Rails has lifecycle methods available in the *model* and *controller* files. In Rails, using these lifecycle methods is optional, just like in React.

In the *model* file, there's lifecycle methods available before/after save. In the *controller* files, there's lifecycle methods available before and after a particular controller function is called.

Another piece of "magic" that [NYC Tours](https://github.com/jeffthemaximum/nyc_tour_backend) uses is that all files are loaded up into memory when the server starts - you don't need a bunch of `require` or `import` statements at the top of a file. Just about every file, class, and function (with some exceptions) is just globally available.

### an aside about ruby

#### return
One piece of Ruby weirdness which you'll see is functions don't need a `return` statement. So if I defined a ruby function like this:

```
def sum(a, b)
  a + b
end
```

That function will just return whatever is the last line of the function, in this case, the result of `a + b`. You can add `return` if you want, it's optional, so this also works:

```
def sum(a, b)
  return a + b
end
```

#### parenthesis
Another piece of weirdness is you don't need parentheses around arguments you pass to a function when you call it. So I can call the above function like:

```
sum(1, 2)
```

And it would return `3`, or I could call it like:

```
sum 1, 2
```

And it'll also return 3.

#### unless
Something like this:

```
foo = false
if (!foo)
	puts 'hi'
end
```

is valid in ruby, and will `put`, or print, `hi`. You can rewrite it with `unless`, like:

```
foo = false
unless (foo)
	puts 'hi'
end
```

And this will also print `hi`. `unless` is like `if not`.

### lower-level overview

Let's trace a request through the POST `api/v1/user` endpoint to see how this works.

- The first file that gets called is `routes.rb`. You can see the definition of POST `api/v1/user` [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/config/routes.rb#L8). That syntax tells rails to call the `create` function in the `users_controller.rb` file.
- That `create` function is then executed by Rails, which is [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L6-L14). The lifecycle methods that I mentioned above come into play here. and it's a little tricky, so I'll do my best to describe:
	- You can see [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L4) we use `skip_before_action`.  That syntax says to not do the `before_action` of calling `verify_authorized` for the `create` controller.
	- `before_action` and `skip_before_action` are built-in Rails controller lifecycle methods.
	- With [this syntax](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L3), the `UsersController` class inherits from the `ApiController` class.
	- `ApiController` is [where we're using](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api_controller.rb#L4-L5) `before_action` to call `verify_authorized`
	- `verify_authorized` is defined [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/lib/auth_util.rb#L8).
	- To summarize, the `users#create` function skips calling the built in Rails `before_action` lifecycle method, which would otherwise do the logic in `verify_authorized`. That's the best I could do at describing it.
- [users#create](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L6-L14) controller is executed.
	- The combination of lines [7](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L7) and [22-24](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L22-L24) create a new `User` model, and tell the controller to use the `email,  password,  password_confirmation` fields from the data that's sent to this endpoint.
	- `user.save` executes the lifecycle methods in the `User.rb` model, and returns false if there's any errors.
	- The lifecycle methods are defined [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/models/user.rb#L17-L21).
	- `before_validation`, `before_save`, and `validates` are built in lifecycle methods here. The order they're executed is `before_validation` -> `validates` -> `before_save`. More info on these [here](https://guides.rubyonrails.org/active_record_callbacks.html#available-callbacks).
- users#create returns a JSON reponse.
	- That happens [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/controllers/api/v1/users_controller.rb#L13).
	- `UserSerializer`, which defines the shape of the JSON response, is defined [here](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/serializers/user_serializer.rb).
	- [These lines](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/serializers/user_serializer.rb#L4-L5) define the top level keys in that JSON response, where the values are attributes from the [User model](https://github.com/jeffthemaximum/nyc_tour_backend/blob/51ebbbf50f919b16e95d38ce883ce66def03968d/app/models/user.rb) or methods in the serializer itself.

### The end

I tried to make this relatively short, in hopes that it'd be a starting point for understanding how [NYC Tour Backend](https://github.com/jeffthemaximum/nyc_tour_backend) works. I'd love to hear which parts help you and which parts you have questions about. No matter what, you should try to clone the repo, start up the server, and try to make some changes. Then, I'll do my best to follow up with some answers!
