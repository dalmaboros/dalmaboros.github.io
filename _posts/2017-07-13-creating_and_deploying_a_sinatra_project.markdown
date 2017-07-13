---
layout: post
title:  "Creating and Deploying a Sinatra Project"
date:   2017-07-13 13:02:49 -0400
---


## Introduction
I created a Sinatra-based web application for my end-of-unit Sinatra Portfolio Project. It is a content management system (CMS) for Sickbay, an ongoing series of live alternative music events in Lafayette, Louisiana. The app is designed to allow the organizer of Sickbay to easily create, read, update, and delete shows and news items, as well as artists and venues.

I deployed the app to [Heroku](http://www.heroku.com), and it is viewable at [http://www.sickbay.party](http://www.sickbay.party).

This video walkthrough demonstrates how to use the app:

<iframe width="560" height="315" src="https://www.youtube.com/embed/QB-sK-UujMI" frameborder="0" allowfullscreen></iframe>

## Planning The Project
Before writing any code, I created an outline of the project's models and their associations:

- Models: 
   - `Show`, `Artist`, `Venue`, `News`, and `User`
- Associations:
   - A `show` has many `artists` and an `artist` has many `shows` (many to many)
   - A `show` belongs to a `venue`, and a `venue` has many `shows` (belongs to, has many)
   - A `venue` has many `artists` through `shows`, an `artist` has many venues through `shows` (has many through)
   - A `user` and a `news` item have no relationships with other models

I created a `show_artists` join table and `ShowArtists` model to support the many to many relationship between `shows` and `artists`

I then considered the functionality I wanted the application to have. A logged-in user should be able to create, read, update, and delete instances of all models. When not logged in, a visitor should only be able to read a listing of shows and news items. 

- There is a total of 15 forms in this project:
   - Create show, edit show, delete show
   - Create artist, edit artist, delete artist
   - Create venue, edit venue, delete venue
   - Create news, edit news, delete news
   - Create user, edit user, delete user

Since creating show listings is the main feature of the application, the form to create a show is a complex form. A user can create a new venue and one or multiple new artists when creating a new show.

<img src="https://www.dropbox.com/s/r82w7stq7bdj95d/Screenshot%202017-07-13%2010.47.09.png?raw=1" alt="Screenshot of the 'create show' form" width="100%">

All other forms are simple, creating and editing or deleting an instance of one specified model.

## Creating A Sinatra App From Scratch
Once the project plan was laid out, I needed to initialize the environment from scratch.

1. Create a project directory `sickbay-shows`

2. Initialize a git repository and [add the project to GitHub](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)

3. Create a `Gemfile` containing the following code:

   ```
   source 'http://rubygems.org'
   
   gem 'sinatra'
   ```
	 
4. Run `$ bundle install` in the terminal's command line (this creates the `Gemfile.lock` file).

5. Create `config.ru` file in the main directory, containing the following code (grabbed from a previous Sinatra lab):

   ```
   require './config/environment'
   
   if ActiveRecord::Migrator.needs_migration?
     raise 'Migrations are pending. Run `rake db:migrate` to resolve the issue.'
   end
   
   use Rack::MethodOverride
   use ShowsController
   use ArtistsController
   use VenuesController
   use NewsController
   use UsersController
   run ApplicationController
   ```
	 
6. Create the `config/environment.rb` file (again, grabbing code from a previous Sinatra lab):
   
	 ```
   ENV['SINATRA_ENV'] ||= "development"

   require 'bundler/setup'
   Bundler.require(:default, ENV['SINATRA_ENV'])
   
   ActiveRecord::Base.establish_connection(
     :adapter => "sqlite3",
     :database => "db/#{ENV['SINATRA_ENV']}.sqlite"
   )
   
   require_all 'app'
   ```
	 
7. Add gem dependencies in the `Gemfile`:
   
	 ```
   gem 'sinatra'
   gem 'activerecord', :require => 'active_record'
   gem 'sinatra-activerecord', :require => 'sinatra/activerecord'
   gem 'i18n'
   gem 'bcrypt'
   gem 'pry'
   gem 'rack-flash3'
   gem 'rake'
   gem 'require_all'
   gem 'shotgun'
   gem 'sqlite3'
   gem 'thin'
   ```
	 
8. Create the MVC paradigm (models and their associations, respective controllers, and views):

   ```
   ├── app
   │   ├── controllers
   │   │   ├── application_controller.rb
   │   │   ├── artists_controller.rb
   │   │   ├── news_controller.rb
   │   │   ├── shows_controller.rb
   │   │   ├── users_controller.rb
   │   │   └── venues_controller.rb
   │   ├── models
   │   │   ├── concerns
   │   │   │   └── slugifiable.rb
   │   │   ├── artist.rb
   │   │   ├── news.rb
   │   │   ├── show_artist.rb
   │   │   ├── show.rb
   │   │   ├── user.rb
   │   │   └── venue.rb
   │   └── views
   │       ├── artists
   │       │   ├── artist_index.erb
   │       │   ├── create_artist.erb
   │       │   ├── edit_artist.erb
   │       │   └── show_artist.erb
   │       ├── news
   │       │   ├── create_news.erb
   │       │   ├── edit_news.erb
   │       │   └── news_index.erb
   │       ├── shows
   │       │   ├── create_show.erb
   │       │   ├── edit_show.erb
   │       │   ├── not_found.erb
   │       │   ├── show_show.erb
   │       │   └── shows_index.erb
   │       ├── users
   │       │   ├── create_user.erb
   │       │   └── edit_user.erb
   │       ├── venues
   │       │   ├── create_venue.erb
   │       │   ├── edit_venue.erb
   │       │   ├── show_venue.erb
   │       │   └── venues_index.erb
   │       ├── backdoor.erb
   │       ├── dashboard.erb
   │       ├── layout.erb
   │       └── login.erb
   ```

9. Create a `Rakefile` containing the following code (grabbed from a previous Sinatra lab):

   ```
   ENV["SINATRA_ENV"] ||= "development"
   
   require_relative './config/environment'
   require 'sinatra/activerecord/rake'
   
   # Type `rake -T` on your command line to see the available rake tasks.
   
   task :console do
     Pry.start
   end
   ```

8. Create migrations in `db/migrate` directory:

   ```
   create_table :shows do |t|
      t.string :date
      t.integer :venue_id
      t.string :url
      
      t.timestamps null: false
   end
   
   create_table :artists do |t|
      t.string :name
      
      t.timestamps null: false
   end
		
   create_table :venues do |t|
      t.string :name
      
      t.timestamps null: false
   end
   
   create_table :show_artists do |t|
      t.integer :show_id
      t.integer :artist_id
      
      t.timestamps null: false
   end
   
   create_table :users do |t|
      t.string :username
      t.string :email
      t.string :password_digest
   end
   
   create_table :news do |t|
      t.string :date
      t.string :content
      t.string :url
   
      t.timestamps null: false
   end
   ```

9. Run migrations with `$ rake db:migrate`

10. Create `db/seed.rb` file, and seed database with `$ rake db:seed` (optional, but helpful).

11. Write controllers and views.

## Deploying A Sinatra App On Heroku

As a web site created for a client, my Sinatra Portfolio Project needed to be useable. [This article](https://devcenter.heroku.com/articles/getting-started-with-ruby-o) from the Heroku documentation is a helpful guide on how to deploy an app on their platform. In brief, you must:

1. [Create an account on Heroku](https://signup.heroku.com/dc)

2. [Download the Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install)

3. Log in to Heroku with your credentials from the command shell:

   ```
   $ heroku login
   ```

4. Add `ruby 2.3.0` (or desired version) to the `Gemfile`.

5. Create a `Procfile` in the project's root directory with the following code: 

   ```
   web: bundle exec rackup config.ru -p $PORT`
   ```

6. Make sure to switch from SQLite3, which is unsupported by Heroku, to PostgreSQL (two helpful guides: [1](http://recipes.sinatrarb.com/p/databases/postgresql-activerecord), [2](https://devcenter.heroku.com/articles/sqlite3))

7. Push your project to your remote git repository

8. Deploy your app to Heroku!

   ```
   $ heroku create
   $ git push heroku master
   ```

9. Instruct Heroku to execute a `web` dyno process type:

   ```
   $ heroku ps:scale web=1
   ```

10. Visit your app:

   ```
   $ heroku open
   ```

Ta da!

Hint: I'm no Heroku master (heck, I won't even claim to understand it), but I have found that scaling down to zero dynos, then back up to one, has been helpful in resolving glitches on my app:

   ```
   $ heroku ps:scale web=0
   $ heroku ps:scale web=1
   ```

## Resources
- Sickbay CMS Sinatra Portfolio Project: http://www.sickbay.party
- GitHub repository: https://github.com/dalmaboros/sickbay-shows
- Video walkthrough: https://www.youtube.com/watch?v=QB-sK-UujMI
