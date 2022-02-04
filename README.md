# WordGuesser on Rails

**NOTE: Do not clone this repo to your workspace. Fork it first, then clone your fork.**

In a [previous
assignment](https://github.com/saasbook/hw-sinatra-saas-wordguesser) you
created a simple Web app that plays the WordGuesser game.

More specifically:

1. You wrote the app's code in its own class, `WordGuesserGame`, which
knows nothing about being part of a Web app.

2. You used the Sinatra framework to "wrap" the game code by providing a
set of RESTful actions that the player can take, with the following routes:

* `GET  /new`-- default ("home") screen that allows player to start new game
* `POST /create` -- actually creates the new game
* `GET  /show` -- show current game status and let player enter a move
* `POST /guess` -- player submits a letter guess
* `GET  /win`   -- redirected here when `show` action detects game won
* `GET  /lose`  -- redirected here when `show` action detects game lost

3. To maintain the state of the game between (stateless) HTTP requests,
you stored a copy of the `WordGuesserGame` instance itself in the
`session[]` hash provided by Sinatra, which is an abstraction for
storing information in cookies passed back and forth between the app and
the player's browser.

In this assignment, you'll reuse the *same* game code but "wrap" it in a
simple Rails app instead of Sinatra.

## Learning Goals

Understand the differences between how Rails and Sinatra handle
various aspects of constructing SaaS, including: 

* how routes are defined and mapped to actions; 
* the directory structure used by each framework;
* how an app is started and stopped; 
* how the app's behavior can be inspected by looking at logs or invoking
a debugger. 


# 1. Run the App

**NOTE: You may find these [Rails guides](http://guides.rubyonrails.org/v4.2/) and the 
[Rails reference documentation](http://api.rubyonrails.org/v4.2.9/) 
helpful to have on hand.**

Like substantially all Rails apps, you can get this one running by doing
these steps:

1. Clone or fork the repo

1. Change into the app's root directory `hw-rails-wordguesser`

1. Run `bundle install --without production`

1. | Local Development                      	| Codio                                                     	|
    |----------------------------------------	|-----------------------------------------------------------	|
    | Run `rails server` to start the server 	| Run <br>`rails server -b 0.0.0.0`<br> to start the server 	|

**Q1.1.**  What is the goal of running `bundle install`?

**A:** Install all the dependencies

**Q1.2.**  Why is it good practice to specify `--without production`
when running 
it on your development computer?

**A:** Because you avoid installing gems you will not use while developing the application

**Q1.3.** 
(For most Rails apps you'd also have to create and seed the development
database, but like the Sinatra app, this app doesn't use a database at all.)

Play around with the game to convince yourself it works the same as the
Sinatra version.

> To view the game in Codio, use the Preview button that says "Project Index" in the top tool bar. Click the drop down and select "Box URL" 
>
> ![BoxURLpreview](https://global.codio.com/content/BoxURLpreview.png)
>
> For subsequent previews, you will not need to press the drop down -- your button should now read "Box URL".

# Code Comprehension Questions

## 2. Where Things Are

Both apps have similar structure: the user triggers an action on a game
via an HTTP request; a particular chunk of code is called to "handle"
the request as appropriate; the `WordGuesserGame` class logic is called
to handle the action; and usually, a view is rendered to show the
result.  But the locations of the code corresponding to each of these
tasks is slightly different between Sinatra and Rails.


**Q2.1.** Where in the Rails app directory structure is the code corresponding
to the `WordGuesserGame` model?

**A:** `./app/models/wordguesser_game.rb`

**Q2.2.** In what file is the code that most closely corresponds to the 
logic in the Sinatra apps' `app.rb` file that handles incoming user
actions?

**A:** `game_controller.rb`

**Q2.3.** What class contains that code?

**A:** `GameController`

**Q2.4.** From what other class (which is part of the Rails framework)
does that class inherit?

**A:** `ApplicationController`

**Q2.5.** In what directory is the code corresponding to the Sinatra app's views
(`new.erb`, `show.erb`, etc.)?

**A:** `./app/views`

**Q2.6.** The filename suffixes
for these views are different in Rails than they were in the Sinatra
app.  What information does the rightmost suffix of the filename 
(e.g.: in `foobar.abc.xyz`, the suffix `.xyz`) tell
you about the file contents?

**A:** It tells Rails to process the Ruby code in the file 

**Q2.7.** What information does the  other suffix tell you about what
Rails is being asked to do with the file?

**A:** The other suffix tells Rails to generate an HTML page 
with contents dynamically added after processing the Ruby code

**Q2.8.** In what file is the information in the Rails app that maps
routes (e.g. `GET /new`)  to controller actions?  

**A:** `./config/routes.rb`

**Q2.9.** What is the role of the `:as => 'name'` option in the route
declarations of `config/routes.rb`?  (Hint: look at the views.)

**A:** This option will allow the automatic creation of helper 
methods like 'name'_path that returns a specific URI

## 3. Session

Both apps ensure that the current game is loaded from the session before
any controller action occurs, and that the (possibly modified) current
game is replaced in the session after each action completes.

**Q3.1.** In the Sinatra version, `before do...end` and `after do...end` blocks
are used for session management.  What is the closest equivalent in this
Rails app, and in what
file do we find the code that does it?

**A:** In this app this is done using the `before_action :get_game_from_session` 
and `after_action  :store_game_in_session` filter methods. 
They are located in `app/controllers/game_controller.rb`

**Q3.2.** A popular serialization format for exchanging data between Web
apps is [JSON](https://en.wikipedia.org/wiki/JSON).  Why wouldn't it
work to use JSON instead of YAML?  (Hint: try replacing `YAML.load()`
with `JSON.parse()` and `.to_yaml` with `.to_json` to do this test.  You
will have to clear out your cookies associated with `localhost:3000`, or
restart your browser with a new Incognito/Private Browsing window, in
order to clear out the `session[]`.  Based on the error messages you get
when trying to use JSON serialization, you should be able to explain why
YAML serialization works in this case but JSON doesn't.)

**A:** The game and its methods wouldn't be loaded since the JSON and YAML
syntaxes are different.

## 4. Views

**Q4.1.** In the Sinatra version, each controller action ends with either
`redirect` (which as you can see becomes `redirect_to` in Rails) to
redirect the player to another action, or `erb` to render a view.  Why
are there no explicit calls corresponding to `erb` in the Rails version?
(Hint: Based on the code in the app, can you discern the
Convention-over-Configuration rule that is at work here?)

**A:** Controller actions do not return a value; instead, when a 
controller action finishes executing, by default Rails will identify and render 
a view named `app/views/model-name/action.html.erb`, for example `app/views/game/show.html.erb`
for the show action in GameController

**Q4.2.** In the Sinatra version, we directly coded an HTML form using the
`<form>` tag, whereas in the Rails version we are using a Rails method
`form_tag`, even though it would be perfectly legal to use raw HTML
`<form>` tags in Rails.  Can you think of a reason Rails might introduce
this "level of indirection"?

**A:** If you use Rails form helpers, the field names are chosen such that
params[:model] is a hash whose keys and values are the user-supplied values for
an instance of model.

**Q4.3.** How are form elements such as text fields and buttons handled in
Rails?  (Again, raw HTML would be legal, but what's the motivation
behind the way Rails does it?)

**A:** By using form helpers to define fields it becomes easier to parse
the parameters needed by the action.

**Q4.4.** In the Sinatra version, the `show`, `win` and `lose` views re-use the
code in the `new` view that offers a button for starting a new game.
What Rails mechanism allows those views to be re-used in the Rails version?  

**A:** The `render` method.

## 5. Cucumber scenarios

The Cucumber scenarios and step definitions (everything under
`features/`, including the support for Webmock in `webmock.rb`) was
copied verbatim from the Sinatra version, with one exception: the
`features/support/env.rb` file is simpler because the `cucumber-rails`
gem automatically does some of the things we had to do explicitly in
that file for the Sinatra version.

Verify the Cucumber scenarios run and pass by running `rake cucumber`.

**Q5.1.** What is a qualitative explanation for why the Cucumber scenarios and
step definitions didn't need to be modified at all to work equally well
with the Sinatra or Rails versions of the app?

**A:** Because the logic behind the apps is the same, they have the same pages
with the same elements with the same behaviors.

