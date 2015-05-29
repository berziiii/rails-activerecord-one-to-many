![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

## Objectives
* Use the resources method in the routes file to generate all the routes.
* Create a Review Model and Migration that implements a move review.
* Review the migration that implements this relationship.
* Draw an Entity Relationship Diagram (ERD) to show how foreign keys are used to implement these 1 to many relationships in the DB.
* Use ActiveRecord `has_many` and `belongs_to` to implement this movie/review relationship.
* Use the Rails console to create movie reviews.
* Create seed data to pre-populate a couple of movie reviews.
* Create a Review's controller that will return a JSON representation of movie reviews.
* Create a nested resource for movie reviews.

## Code Along: Setup

**Make sure you fork, clone, create the DB, migrate and seed the DB.**

```
rake db:create
rake db:migrate
rake db:seed
```

**Run the server**

```
rails server
```

Ok, you should now be able to see the JSON for all three movies at `http://localhost:3000`.

## Code Along: Create movies routes using 'resources'

Previously, we were creating routes for each Movie Controller action. *This is tedious.* Let's see how we can make this more concise.

**But, first lets look at all of routes!**

```
rake routes
```

**Change config/routes to**

```ruby
Rails.application.routes.draw do
  # Default root for '/' in this application                                    
  root 'movies#index'

  # create routes for movie resource                                            
  resources :movies, except: [:new, :edit]
end
```

The 'resources' method will automatically generate all the routes we've creating individually. *Much better.*

**In another terminal run rake routes again and compare the routes.**

```
rake routes
```

## Code Along: Create a Review Model

Lets use a rails generator to create the review model. This will also create a migration for this model.

**Create the Review Model and Migration. Apply the migration**
```
rails g model Review name comment:text movie:references
rake db:migrate
```

**Open up the migration generated**

This will show ,for us, a new kind of column. A column that will contain the **foreign key** to a movie.

```ruby
  ...
  t.references :movie, index: true, foreign_key: true
  ...
```

This will create a **foreign key** column in the reviews table. 

The **foreign key** column will contain the id of movie that the review pertains to. 

Another words, the movie "Affliction" has an id of 1. If we create a Review for this movie the row in the DB for this review will have a **foreign key** column, named *movie_id*, with a value of 1.

**Open up the Rails DB console and take a look at the tables we now have**

```
rails db

\dt
\d movies
\d reviews
```

The `\d` will show all the tables.

 `\dt movies` and `\dt reviews` will show the columns for movies and reviews tables. 
 
*Notice that the reviews table now has a movie_id column. This is the foreign key that references a specific movie by it's id.*

**Open up the db/schema.rb**

And see that the 'reviews' table has a entry 'movie_id' that is the **foreign key**.


## Code Along: Create a Movie Review.

**Open the app/models/review.rb**

Notice the **belongs_to** method in the Review Model. This will create a relationship from the review to the movie that the review pertains to.

**Add this to the app/models/movies.rb**

```
has_many :reviews
```

The *has_many* method will create a relationship from the Movie Model to the Review model.

Notice how the langauge matches the relationships. A movie may **have many** reviews. And a review **belongs to** one movie.

This is called a **one to many** relationship. Each movie can have many reviews.

**Enough talk, let's create the review in Rails console.**

```
rails c

m1 = Movie.first
m1.reviews

```

Notice, a movie now has the reviews method! The **has_many** added to the Movie Model above actually gave the Movie Model this reviews method.

If we were to do this by hand, not using **has_many**, we would do **something like** this. *Warning: very, very simplified example!*

```
 class Movie < ActiveRecord::Base
 
   # VERY SIMPLIFIED EXAMPLE ONLY, DON'T DO THIS!!
   def reviews
      # return the Array of reviews for this movie
      reviews 
   end
 end
```
Let's continue.

```
Review.all

m1.reviews.create(name: 'Tom', comment: 'Dark, somber')

r1 = Review.first

m1.reviews
```

First we see that there are no Reviews when we run `Reviews.all`. 

Then we create a review for the movie 'Affliction' with `m1.reviews.create(name: 'Tom', comment: 'Dark, somber')`. 

Then we get the one and only review at this time in the DB with the `r1 = Review.first`. 

Then we show all the movie, 'Affliction', reviews, `m1.reviews`.

**Let's draw what the movies and reviews tables in the DB look like at this time**

**Let's confirm our drawing by using rails db**

```
rails db

SELECT * FROM reviews;

SELECT * FROM movies;
```

See how the movie_id in the reviews column has the value 1. This is the id of the movie that the review is for.


### Lab: Create a Album with Songs.
Work in Groups.

* An Album will have a title, artist name, released year.

* A Song will have a title, duration and price.

* An Album may have many Songs.

* A Song belongs to an Album.

* Create, in the Rails console, a couple of Albums. Each having one or more Songs. 

* (Optionally) Create Albums and Songs in the seed file.

* What happens if in the rails console you:

```
s1 = Song.first
s1.album
```

Yes, the `belongs_to` in the Song Model adds a `album` method to the Song that returns the album the song belongs to.

* Draw, as a group, the DB tables for Albums and Songs. Each table should have a row for each Album and Song. (Don't forget to show the foreign keys!)

## Code Along: Populate Movie Reviews

**Add to the seed file**

```
Review.delete_all
Movie.delete_all

movie = Movie.create!(name: 'Affliction', rating: 'R', desc: 'Little Dark', len\
gth: 123)
movie.reviews.create!(name: 'Tom', comment: 'Dark, somber')
movie.reviews.create!(name: 'Meg', comment: 'Slow, boring')

movie = Movie.create!(name: 'Mad Max', rating: 'R', desc: 'Fun, action', length\
: 154)
movie.reviews.create!(name: 'Joe', comment: 'Explosions, silly')
movie.reviews.create!(name: 'Christine', comment: 'Brilliant, fun')

movie = Movie.create!(name: 'Rushmore', rating: 'PG-13', desc: 'Quirky humor', \
length: 105)
movie.reviews.create!(name: 'Tom', comment: 'Crazy, humor')
movie.reviews.create!(name: 'Joanne', comment: 'Waste of time, stupid')

puts "Created three Movies"
```

## Code Along: Create a nested route for Reviews.

When we access a review we need to **ALWAYS** refer to the movie that that review belongs to. 


For example when we want to see all a specific movie's reviews we will use the URL. `http://localhost:3000/movies/1/reviews`

**Add this to the routes file**  

```ruby
  # create routes for movie resource                                            
  resources :movies, except: [:new, :edit] do
    # create nested routes for the movie reviews                                
    resources :reviews, except: [:new, :edit]
  end

```

**Run rake routes to look at the new routes created**

## Code Along: Create a Reviews Controller.

**Create a reviews controller** 

```
class ReviewsController < ApplicationController
  # Execute this method before each action is executed                          
  before_action :set_movie

  # GET /movies/:movie_id/reviews                                               
  def index
    # all the movies                                                            
    @reviews = @movie.reviews
    render json: @reviews
  end

  private

  # find the movie for the review/s                                             
  def set_movie
    # create an instance variable that can be accessed in                       
    # every action.                                                             
    @movie = Movie.find(params[:movie_id])
  end
end

```

This will use the `before_action` method that will find the movie that the reviews belong to. *Look up the before_action method in the Rails documentation.*


