# Photogram Industrial Part 4: Profile page

## Walkthrough video

<div class="bg-red-100 py-1 px-5" markdown="1">
**Please note**, the video is from a previous iteration of the project, so there are some differences:

- I am using Gitpod as my cloud editor, so the interface looks a bit different.
- Anything contained in the project "README" is now contained in this Lesson
- I use `bin/server` to start my live app preview, _you_ should use `bin/dev`
- I use `rails sample_data`, _you_ should use `rake sample_data`
- I am using using Bootstrap version 4 in the video. The official Bootstrap docs now show v5 by default. Links and code contained in this document are based on the version 5 Bootstrap.
</div>

Did you read the differences above? Good! Then [here is a walkthrough video for this project.](https://share.descript.com/view/h3WXOoqhNNU)

**As you watch the video, pause it frequently, read the associated text, and type out the code.**

## Getting started

Let's continue with `photogram-industrial`, keeping in mind a rough target to work towards:

[photogram-industrial.matchthetarget.com](https://photogram-industrial.matchthetarget.com/)

So navigate to `github.com/codespaces` and reopen your `photogram-industrial` codespace to continue building on what you accomplished in _Photogram Industrial Parts 1, 2, and 3_.

<div class="bg-red-100 py-1 px-5 bleed-full" markdown="1">

By the end of this lesson, you should: 

- Run `rake grade` and get full credit for the `photogram-industrial` automated tests. (Additional grading on this project will come from the pull requests.)

- Submit four pull requests (PRs) on Canvas:
  - `<initials>-photogram-industrial` compared to `main`
  - `<initials>-starting-on-ui` compared to `<initials>-photogram-industrial`
  - `<initials>-profile-page` compared to `<initials>-starting-on-ui`
  - `<initials>-tabbed-interface` compared to `<initials>-profile-page`
</div>

## Branch and open pull requests

Before we begin, I'd like to create a new branch. First, let's make sure to open pull requests on Github for the branches we've been working on.

<div class="bg-blue-100 py-1 px-5" markdown="1">

Follow along with the video for a visual demonstration of the process here.
</div>

Make sure you've pushed the `<initials>-photogram-industrial` and `<initials>-starting-on-ui` branches to Github and that `git status` (or, as we aliased it, just `g`) returns `Everything up-to-date`. 

Navigate to your GitHub repository fork of the project (where you have been pushing to at `github.com/YOUR_USERNAME/photogram-industrial`). If you switch to one of your branches, you can click a button to open the pull request.

At this point, you can compare the branch to any of the other branches or commits. I could compare it to the `appdev-projects/photogram-industrial` parent repository that we forked it from. That's not always the case because most of the time when you work on your own projects, it will be from a brand new repository and not a fork of somebody elses's repository.

In this case, we will switch the **base repository** to my _own_ fork of the repository (e.g., `raghubetina/photogram-industrial`), and compare our `<initials>-photogram-industrial` against the **base branch** `main`.

With that selected, we can pick a nice name for the pull request. Don't just use the branch name, add some explanation. Perhaps: "Setting up domain model, business logic, and sample data". Now you can open the PR.

Within the PR we can now view the files that were changed in line-by-line granular detail (generated by `git diff`), and we can receive comments and review of those changes. Any more commits that we make on the branch will be automatically updated on this open PR!

With a PR open on `<initials>-photogram-industrial` to `main`, we can open a second PR for the `<initials>-starting-on-ui` branch. Again, switch the base repository to your own repo. In this case, we _don't_ want to compare this second branch to `main`, because that would take all of the changes including those from the branch `<initials>-photogram-industrial` that we branched off (and just opened a PR for). Instead, we will compare `<initials>-starting-on-ui` to _its_ parent branch `<initials>-photogram-industrial`.

Again, we can give this PR another useful title, like "Starting on UI", then open it. And we will see the `git diff` in this PR just compares the two branches, ignoring `main`.

Back in a terminal, make sure you are on the `starting-on-ui` branch (`g co <branch-name>`), and branch off of _this_ branch to continue your work.

```
% git co rb-starting-on-ui
% git cob rb-user-profile
```

## User show page route

Let's get the app running with `bin/dev`. 

I want to start on the user's show page, because everything will hinge on that. There will be links there for their liked photos, their feed, and their discover page. We'll also need a way to follow users on that page. 

Typically, the URL for a user's page is just `/username`, like `/alice`. Of course, if you try that now you will get a `No route matches` error.

Devise doesn't set up the normal CRUD routes for us, so even if we tried something like `/users/1`, we would still get the routing error.  

But that's okay, because we know how to RCAV. We can build the route we want from scratch, starting with `routes.rb`:

```ruby{6}
# config/routes.rb

Rails.application.routes.draw do
  root "photos#index"
  
  get "/users/:id" => "users#show", as: :user

  devise_for :users
  
  resources :comments
  resources :follow_requests
  resources :likes
  resources :photos
end
```

We are using our new shorthand syntax to create this get request, and name it with `as:` so that we have our `user_path` and `user_url` route helper methods. 

We could have just put `resources :users`, but we want more control over things in this case, so we just generate the single route of interest. 

However, there's a variation of `resources`, passing the `only:` option to provide the routes that we want to create:

```ruby{6,14}
# config/routes.rb

Rails.application.routes.draw do
  root "photos#index"
  
  # get "/users/:id" => "users#show", as: :user

  devise_for :users
  
  resources :comments
  resources :follow_requests
  resources :likes
  resources :photos
  resources :users, only: :show
end
```

We'll use the route defined in this way with `resources ... only:`, and put it at the bottom of the list of resources.

But wait! We just defined the route `/users/alice`! What we intended, was the route `/alice` (i.e., `/username`).

In that case, what we actually want is just:

```ruby{13}
# config/routes.rb

Rails.application.routes.draw do
  root "photos#index"

  devise_for :users
  
  resources :comments
  resources :follow_requests
  resources :likes
  resources :photos

  get "/:username" => "users#show", as: :user
end
```

Because `"/:username"` is such a general route we need to put it at the bottom of the list. Otherwise, anytime we tried to get to `"/photos` or `"/comments"`, Rails would try to fill that segment in as the flexible `:username` parameter.

In the live app, try to navigate to `/alice`. What do you know, an error `uninitialized constant UsersController`. Devise didn't generate any `users_controller.rb` for us, so we'll need to make that file and fill it in ourselves:

```ruby
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  def show
  end
end
```

We don't need the `render` statement, because we will right away make our conventionally named view template `app/views/users/show.html.erb`, say "hi" in it, and try to view the page `/alice` again. Working? Good! Commit that work.

Continuing on, we want to back into the logic we need to make the action do what we want:

```ruby{5}
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  def show
    @user = User.find_by(username: params.fetch(:username))
  end
end
```

We need the `find_by` method, because the value returned by `params.fetch(:username)` in our example is `"alice"`, which won't work with `find`, because that method expects an ID number.

Make sure that worked by adding something to the view template:

```erb
<!-- app/views/users/show.html.erb -->

<h1>
  <%= @user.username %>
</h1>
```

And refresh `/alice` to see the result. What if we change the URL to something that doesn't exist, like `/foo`?

If we were using the `find` method, the return would be an `record not found` error. That's good, because it would mean a 404 page for the user, rather than an internal 500-type server error on our side. But, as we see when we try to navigate to the non-existent user, `find_by` returns the more problematic `undefined method for nil` 500-type error that we want to avoid. So what to do?

We want a 400 error, which are errors caused by normal operation, like someone trying to navigate to a location that doesn't exist. 

There's actually a variation of `find_by` called `find_by!` with an exclamation mark. The purpose of that is to throw the `record not found` error:

```ruby{5:(18-25)}
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  def show
    @user = User.find_by!(username: params.fetch(:username))
  end
end
```

Now `/alice` should still work, and `/foo` should show the 400-type error.

Commit!

```
g acm "Set up users#show action"
```

And push! (with the `--set-upstream` flag if it's the first push)

```
g p
```

Once you push, you should see a link to open a pull request in the terminal. You can click that link, and then be sure to set the base branch to the most recent branch (`rb-starting-on-ui` for me), so you are only comparing these recent changes to that (which is in turn comparing against `photogram-industrial`, which is in turn comparing against `main`).

## User profile own photos

With our route working, data marshalled in the controller, and view template waiting for us, let's begin to add some content.

We'll start by getting some photos on the page, then worry about making them look pretty:

```erb{7-16}
<!-- app/views/users/show.html.erb -->

<h1>
  <%= @user.username %>
</h1>

<h2>Own photos</h2>

<ul>
  <% @user.own_photos.each do |photo| %>
  <li>
    <%= photo.caption %>
    <img src="<%= photo.image %>">
  </li>
  <% end %>
</ul>
```

Since we spent so much time in the backend, we have lots of methods like `.own_photos` that make this process painless! Refresh `/alice` and see the result.

Back to the view template, there's actually, you guessed it, a helper method for `<img>` that we should be using instead of the HTML element:

```erb{8}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
<ul>
  <% @user.own_photos.each do |photo| %>
  <li>
    <%= photo.caption %>
    <%= image_tag photo.image %>
  </li>
  <% end %>
</ul>
```

There's more option that `image_tag` can take to get different formats back, among other things, but we'll just use it as-is for now.

Bootstrap has [a bunch of different types of cards](https://getbootstrap.com/docs/5.2/components/card/), including some for exactly what we want here: an image, some information about the thing, and comments. We can use [the kitchen sink example](https://getbootstrap.com/docs/5.2/components/card/#kitchen-sink) as a good jumping off point.

As usual, we can copy in the example and style it in our view template:

```erb{7-27}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
<h2>Own photos</h2>

<% @user.own_photos.each do |photo| %>
  <div class="row mb-4">
    <div class="col-md-6 offset-md-3">
      <div class="card">
        <%= image_tag photo.image, class: "card-img-top" %>
        <div class="card-body">
          <h5 class="card-title"><%= photo.owner.username %></h5>
          <p class="card-text"><%= photo.caption %></p>
        </div>
        <ul class="list-group list-group-flush">
          <li class="list-group-item">An item</li>
          <li class="list-group-item">A second item</li>
          <li class="list-group-item">A third item</li>
        </ul>
        <div class="card-body">
          <a href="#" class="card-link">Card link</a>
          <a href="#" class="card-link">Another link</a>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

We made some changes to the example when we put it in our view template. Those were as follows:
  
  - Put the entire card in a Bootstrap grid `"row"` with a margin bottom `mb` of 4 (to provide some vertical spacing), and `"col"` with a margin distance `md` of 6 and offset of 3 (to center it). 
  - Removed the `sytle="width: 18rem;"` from the `"card"`, since that would have offset our centering. 
  - Replaced the `<img ...>` with our `image_tag`, using the class `"card-img-top"`. 
  - Replaced the dummy copy in the `<h5 class="card-title"></h5>` element with `photo.owner.username`.
  - Replaced the dummy copy in the `<p class="card-text"></p>` element with `photo.caption`. 

Now, rather than each `photo` rendering in an `<li>` within a `<ul>`, they will render in a separate card.

Refresh `/alice` and view the results.

It looks good, but now we want to render the comments in place of "An item", "A second item", etc. below the figure caption. We can do that by creating a nested `.each` loop to render each caption on the card iteratively:

```erb{14-18}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
<% @user.own_photos.each do |photo| %>
  <div class="row mb-4">
    <div class="col-md-6 offset-md-3">
      <div class="card">
        <%= image_tag photo.image, class: "card-img-top" %>
        <div class="card-body">
          <h5 class="card-title"><%= photo.owner.username %></h5>
          <p class="card-text"><%= photo.caption %></p>
        </div>
        <ul class="list-group list-group-flush">
          <% photo.comments.each do |comment| %>
            <li class="list-group-item">
              <%= comment.body %>
            </li>
          <% end %>
        </ul>
        <div class="card-body">
          <a href="#" class="card-link">Card link</a>
          <a href="#" class="card-link">Another link</a>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

The card is looking pretty good, but let's grab the Bootstrap [media objects](https://getbootstrap.com/docs/4.6/components/media-object/) code to give us the option of adding an avatar to each comment:

<div class="bg-red-100 py-1 px-5" markdown="1">

In Bootstrap version 5, media objects now [need to be recreated using flex utilities](https://getbootstrap.com/docs/5.2/utilities/flex/#media-object). The video uses the version 4 media object option, but the code below (and your code) should use the more recent update.
</div>

```erb{16-24}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
<% @user.own_photos.each do |photo| %>
  <div class="row mb-4">
    <div class="col-md-6 offset-md-3">
      <div class="card">
        <%= image_tag photo.image, class: "card-img-top" %>
        <div class="card-body">
          <h5 class="card-title"><%= photo.owner.username %></h5>
          <p class="card-text"><%= photo.caption %></p>
        </div>
        <ul class="list-group list-group-flush">
          <% photo.comments.each do |comment| %>
            <li class="list-group-item">
              <div class="d-flex">
                <div class="flex-shrink-0">
                  <img src="..." alt="...">
                </div>
                <div class="flex-grow-1 ms-3">
                  <h5 class="mt-0"><%= comment.author.username %></h5>
                  <p><%= comment.body %></p>
                </div>
              </div>
            </li>
          <% end %>
        </ul>
        <div class="card-body">
          <a href="#" class="card-link">Card link</a>
          <a href="#" class="card-link">Another link</a>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

For now, the `<img src="...">` will just be a placeholder, since we didn't add profile pictures as a column in the `User` table. But, when we do add these, we'll be able to replace that line and get some avatars for each comment.

Now's a good time to...

```
g acm "Added own photos to users#show"
```

## User profile add comment

The next step is to add a comment form at the bottom of each photo card. Happily, because of the `scaffold` generator, we already have comment forms ready for us as a partial in `app/views/comments/` as `_form.html.erb`, so let's just render those in:

```erb{11-13}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
                  <h5 class="mt-0"><%= comment.author.username %></h5>
                  <p><%= comment.body %></p>
                </div>
              </div>
            </li>
          <% end %>
        </ul>
        <div class="card-body">
          <%= render "comments/form" %>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

Is that going to work? Refresh and find out. 

No, because of an `undefined local variable or method comment` in this partial. The form is expecting us to pass a `comment` as a local variable: 

```erb{5:(36-56)}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
        <div class="card-body">
          <%= render "comments/form", comment: Comment.new %>
        </div>
      </div>
    </div>
  </div>
<% end %>
```

Now we have a comment form below every photo! It doesn't look great though, because it was auto-generated. For instance, we don't want the user to have to type in the "Author" or "Photo" fields. Those IDs should be pre-filled and hidden. The user should only see the "Body" field (with the label hidden as well).

That means we'll need to go into the `comments/_form.html.erb` partial to make some changes. 

Before I make any changes to a partial, I first need to be sure that no one else is using it; as in, no other view templates are relying on it. When we start to use partials, we are coupling our codebase. This is the perfect example of why **automated tests** are so vital. If you had tests covering all of your view templates, you wouldn't need to worry about breaking anything for other people working on the codebase.

<div class="bg-red-100 py-1 px-5" markdown="1">

Actually, there are a few automated tests in the project now in the updated version. Run them with `rake grade`. You may need to go back to the _Photogram Industrial Part 1_ lesson to relaunch the assignment and retrieve your Grades token to enter into this project.

The test we wrote for you are not extensive and shouldn't be relied on for a full refactoring; but they are a nice starting point!
</div>

With that in mind, we should make some tests in the very near future. But until then, we're safe to edit this code that we just generated and are working on solo:

```erb{16-24}
<!-- app/views/comments/_form.html.erb -->

<%= form_with(model: comment) do |form| %>
  <% if comment.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(comment.errors.count, "error") %> prohibited this comment from being saved:</h2>

      <ul>
        <% comment.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <%= form.hidden_field :photo_id %>

  <div>
    <%= form.text_area :body %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

We completely deleted the `:author_id` field, made the `:photo_id` hidden, and removed the `:body` label. Because we want the IDs auto-filled, we can do that in the `comments_controller.rb` in the `create` action:

```ruby{7}
# app/controllers/comments_controller.rb

class CommentsController < ApplicationController
  # ...
  def create
    @comment = Comment.new(comment_params)
    @comment.author = current_user

    respond_to do |format|
      if @comment.save
        format.html { redirect_to comment_url(@comment), notice: "Comment was successfully created." }
        format.json { render :show, status: :created, location: @comment }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @comment.errors, status: :unprocessable_entity }
      end
    end
  end
  # ...
end
```

Note that we can just use the `current_user` method that devise created for us, and assign columns with that. Rails can figure out what column we want to extract and assign.

But how do we get the current photo ID into the form? Have a look at the `render` method in the user's show page. Right now we have:

```erb
<%= render "comments/form", comment: Comment.new %>
```

So `comment` in the form is the `Comment.new` object. We can assign a column value to this object when we instantiate it, using the ID of the current photo (the `.each` loop the comments are nested inside of):

```erb
<%= render "comments/form", comment: Comment.new(photo_id: photo.id) %>
```

Alternatively, we could have written:

```erb
<%= render "comments/form", comment: photo.comments.build %>
```

Lastly, let's style the form with Bootstrap so that it looks good. There are nice ways of doing that with [Bootstrap forms](https://getbootstrap.com/docs/4.6/components/forms/). 

<div class="bg-red-100 py-1 px-5" markdown="1">

In Bootstrap version 5, [the form instructions can now be found here](https://getbootstrap.com/docs/5.3/forms/overview/#overview). The video uses the version 4 form components, but the code below (and your code) should use the more recent update.
</div>

```erb{6-7,11}
<!-- app/views/comments/_form.html.erb -->

<!-- ... -->
  <%= form.hidden_field :photo_id %>

  <div class="mb-3">
    <%= form.text_area :body, class: "form-control" %>
  </div>

  <div>
    <%= form.submit class: "btn btn-outline-secondary btn-block" %>
  </div>
<% end %>
```

We also added a button class onto `form.submit` to make that look a bit nicer. I like to start with just black and white there until I make color decisions later on.

On `/alice`, try out the new form. It should work, and it should redirect you to the show page for that comment. 

Let's fix that in the controller to redirect to the _previous location_. There's a nice method we haven't seen yet called `redirect_back`:

```ruby{11}
# app/controllers/comments_controller.rb

class CommentsController < ApplicationController
  # ...
  def create
    @comment = Comment.new(comment_params)
    @comment.author = current_user

    respond_to do |format|
      if @comment.save
        format.html { redirect_back fallback_location: root_path, notice: "Comment was successfully created." }
        format.json { render :show, status: :created, location: @comment }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @comment.errors, status: :unprocessable_entity }
      end
    end
  end
  # ...
end
```

We provide a fallback argument to the method, in case Rails isn't able to detect what page the user came from.

This is a good time to commit.

```
g acm "Added photo commenting"
```

The next thing we want to do is add the other collections of photos: liked, feed, and discover. We'll add these within a tabbed interface, so that we can click on tabs and select which collection of photos we want to see for the the user.

## Tabbed interface

Before we start on the tabbed interface, let's make a new branch (off of `rb-user-profile`) to work on:

```
% g s
On branch rb-user-profile...

% g cob rb-tabbed-interface
```

Back in the live app, on the page `/alice` that we've been working on, we want tabs that say "Liked photos", "Feed", etc. to move between the different photo collections for our user profile.

In the [Bootstrap navs section](https://getbootstrap.com/docs/5.2/components/navs-tabs/), there's a few interfaces with exactly what we want. I'm going to select the [active pills filled and justified](https://getbootstrap.com/docs/5.2/components/navs-tabs/#fill-and-justify).

Let's plop that in the user profile (inside of a `"row"` and `"col"` class for centering with the photo cards):

```erb
<!-- app/views/users/show.html.erb -->

<div class="row">
  <div class="col md-6 offset-3">
    <h1>
      <%= @user.username %>
    </h1>

    <ul class="nav nav-pills nav-justified">
      <li class="nav-item">
        <a class="nav-link active" href="#">Posts</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Liked photos</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Feed</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Followers</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Following</a>
      </li>
    </ul>
  </div>
</div>

<% @user.own_photos.each do |photo| %>
  <!-- ... -->
```

We should see the tabbed interface on the `/alice` page now, but what do we need for it to actually work?

<div class="bg-red-100 py-1 px-5" markdown="1">

In the video, I put the `/alice/liked` action into the `PhotosController`, you should instead follow along with the code below, where I put the action in the `UsersController`.
</div>

We need to create the URL paths for each of the tabs. For instance, `/alice/feed`, `alice/followers`, etc. Let's begin with `/alice/liked`, or, more generally, `/:username/liked`:

```ruby{13}
# config/routes.rb

Rails.application.routes.draw do
  root "photos#index"

  devise_for :users
  
  resources :comments
  resources :follow_requests
  resources :likes
  resources :photos

  get ":username/liked" => "users#liked", as: :liked

  get ":username" => "users#show", as: :user
end
```

Note that we are able to drop the preceding `/` slash from `:username` for both routes, that isn't necessary, and Rails will build this route from the root path.

Let's add this action now:

```ruby{8-10}
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  def show
    @user = User.find_by!(username: params.fetch(:username))
  end

  def liked
    @user = User.find_by!(username: params.fetch(:username))
  end
end
```

And we can add the view template, `app/views/users/liked.html.erb`, and put some dummy copy in there, then refresh `/alice/liked` and make sure our RCAV is up and running.

Once that page is loading, let's add the proper links (with our helper methods) to the tabbed interface:

```erb{11,14}
<!-- app/views/users/show.html.erb -->

<div class="row">
  <div class="col md-6 offset-3">
    <h1>
      <%= @user.username %>
    </h1>

    <ul class="nav nav-pills nav-justified">
      <li class="nav-item">
        <%= link_to "Posts", user_path(@user.username), class: "nav-link" %>
      </li>
      <li class="nav-item">
        <%= link_to "Liked photos", liked_path(@user.username), class: "nav-link" %>
      </li>
      <!-- ... -->
```

Try out the two links on `/alice` and see that they work. Yes? Good time to commit!

Now the next step is getting the liked photos to show up. Actually, the page we want should look exactly like the user's photos page that we just spent all of that time building out! We could copy everything from `users/show.html.erb` and paste it into `users/liked.html.erb`, but this is the perfect time for a partial!

First, let's take all of the code for a photo card and create a partial for that `app/views/photos/_photo.html.erb`:

```erb
<!-- app/views/photos/_photo.html.erb -->

<div class="row mb-4">
  <div class="col-md-6 offset-md-3">
    <div class="card">
      <%= image_tag photo.image, class: "card-img-top" %>
      <div class="card-body">
        <h5 class="card-title"><%= photo.owner.username %></h5>
        <p class="card-text"><%= photo.caption %></p>
      </div>
      <ul class="list-group list-group-flush">
        <% photo.comments.each do |comment| %>
          <li class="list-group-item">
            <div class="d-flex">
              <div class="flex-shrink-0">
                <img src="..." alt="...">
              </div>
              <div class="flex-grow-1 ms-3">
                <h5 class="mt-0"><%= comment.author.username %></h5>
                <p><%= comment.body %></p>
              </div>
            </div>
          </li>
        <% end %>
      </ul>
      <div class="card-body">
        <%= render "comments/form", comment: photo.comments.build %>
      </div>
    </div>
  </div>
</div>
```

Allowing us to replace the code in the `show.html.erb` view template:

```erb{5}
<!-- app/views/users/show.html.erb -->

<!-- ... -->
<% @user.own_photos.each do |photo| %>
  <%= render "photos/photo", photo: photo %>
<% end %>
```

There's a lot of `photo`s in there. Let's be clear:

  - The first `photos/` is the view template folder nested in `app/views/`
  - The second `photo` refers to the partial in the view template folder: `_photo.html.erb`
  - The third `photo` is the name of the local variable in `_photo.html.erb` that needs to be assigned something.
  - The fourth `photo` is the block variable `ActiveRecord::Relation` object from the `.each` loop that we are passing to the partial

If that's clear to you, then why not put the partial in the `liked.html.erb` template as well! And be sure to change the association accessor to our `liked_photos` method:

```erb{29}
<!-- app/views/users/liked.html.erb -->

<div class="row">
  <div class="col md-6 offset-3">
    <h1>
      <%= @user.username %>
    </h1>

    <ul class="nav nav-pills nav-justified">
      <li class="nav-item">
        <%= link_to "Posts", user_path(@user.username), class: "nav-link" %>
      </li>
      <li class="nav-item">
        <%= link_to "Liked photos", liked_path(@user.username), class: "nav-link" %>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Feed</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Followers</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Following</a>
      </li>
    </ul>
  </div>
</div>

<% @user.liked_photos.each do |photo| %>
  <%= render "photos/photo", photo: photo %>
<% end %>
```

Check the links on `/alice` for "Posts" and "Like photos". It should all be working, which means you should be committing!

On your own, see if you can get the same tabbed interface working for the remaining routes:

```ruby{14-16}
# config/routes.rb

Rails.application.routes.draw do
  root "photos#index"

  devise_for :users
  
  resources :comments
  resources :follow_requests
  resources :likes
  resources :photos

  get ":username/liked" => "users#liked", as: :liked
  get ":username/feed"
  get ":username/followers"
  get ":username/following"

  get ":username" => "users#show", as: :user
end
```

For the `followers` and `following` routes where we aren't showing photos in the interface, you could make a simple UI with a [Bootstrap list group of the relevant users, with links to each](https://getbootstrap.com/docs/5.2/components/list-group).

---
