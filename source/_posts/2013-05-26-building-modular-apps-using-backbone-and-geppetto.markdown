---
layout: post
title: "Building Modular apps using Backbone and Geppetto"
date: 2013-05-26 11:06
comments: true
categories: Backbone Marionette Geppetto
---

When building large multi-modular apps, embedding business logic within your Backbone views or models does not scale well.  Ideally you would want to de-couple your business logic from the view logic. Solving this problem becomes really easy using `Geppetto`.<!-- more -->  

# Prerequisites

To better understand the example in this blog post, it is important to have a good knowledge on the following frameworks:

* Backbone & Marionette
* Backbone ModelBinder
  - Please refer to my previous [post](http://niki4810.github.io/blog/2013/03/02/new-post/) on Backbone ModelBinder
* RequireJS
  - `Cary Landholt` has a really good [screen cast](http://www.youtube.com/watch?v=VGlDR1QiV3A) on requireJs

# Experience the application first

We will be building a simple movie search app. We make use of the [rotten tomatoes developer api](http://developer.rottentomatoes.com/) for fetching the movie details. 
Here is the direct link to the application: [link](http://niki4810.github.io/annotate-sources/Communicating-Via-Geppetto/index.html?) 

# Some more information on the application

The application we are building is a simple search page with `three` views. A `search view` on the left for searching a movie, a `result view` on the right for displaying the search result and a `container view` which holds the search and the result views.

# What functionality does each view provide?

Lets list out the functionality that each view is expected to provide.

* Search View
  - Should accept the movie title as users input.
  - Should  shout out the movie title using an event when the search button is clicked.

* Result View
  - Should keep listening for data.
  - Display the data to the user whenever it gets it.

* Container View 
  - Acts as a dumb container that holds the search and the result view.

# Lets create the view's first

Here are the code snippets for each view

### Search View

```javascript
//search view
define(
	[
	"jquery",
	"underscore", 
	"backbone", 
	"marionette",
	"geppetto",
	"text!src/templates/SearchViewTemplate.html"
	],function(
		$,
		_,
		Backbone,
		Marionette,
		Geppetto,
		SearchViewTemplate
	){
	
	
	var SearchView = Marionette.ItemView.extend({
		template : SearchViewTemplate,
		className : "well span4",
		bindings : {
				"title" : '[name = "title"]'
		},
		events : {
			"click button.searchBtn" : "searchClicked"
		},
		searchClicked : function(e) {
			if(this.model.get("title")){
			this.context.dispatch("performSearchEvent"/*event name*/,{data:this.model}/*event payload*/);
			}else{
				//if title is not set, shake the text input
				//should have a required validator, but this would work
				this.$('[name = "title"]').removeClass().addClass('animated shake');
                var that = this;
				var wait = window.setTimeout(function() {
					that.$('[name = "title"]').removeClass()
				}, 1300); 

			}
		},
		//local variable for model binder
		_modelBinder : undefined,
		initialize : function() {
			 _.bindAll(this);
			//on view initialize, initialize _modelBinder
			this._modelBinder = new Backbone.ModelBinder();
			//save the passed in context locally  such that
			// we can dispatch or listen to events on this context
			this.context = this.options.context;
		},
		close : function() {
			//when view closes, unbind Model bindings
			this._modelBinder.unbind();
		},
		onRender : function() {			  
			this._modelBinder.bind(this.model/*the model to bind*/, 
								   this.el/*root element*/, 
								   this.bindings /*bindings*/ );

		}
	});

	return SearchView;
})

```

### Result View

```javascript
//result view
define([
	"jquery", 
	"underscore", 
	"backbone", 
	"marionette",
	"geppetto", 
	"text!src/templates/ResultViewTemplate.html"
	],
 function(
 	$, 
 	_, 
 	Backbone, 
 	Marionette,
 	Geppetto, 
 	ResultViewTemplate) {
	var ResultView = Marionette.ItemView.extend({
		template: ResultViewTemplate,
		className : "well span6 clearfix",
		bindings : {
				"title" : '[name = "title"]',
				"year" : '[name = "year"]',
				"rated" : '[name = "rated"]',
				"rating" :'[name ="rating"]',
				"poster" :{selector: '[name=poster]',  elAttribute: 'src'}
		},
		initialize : function() {
			  _.bindAll(this);
				//on view initialize, initialize _modelBinder
			this._modelBinder = new Backbone.ModelBinder();
			//save the passed in context locally  such that
			// we can dispatch or listen to events on this context
			this.context = this.options.context;
			this.context.listen(this, "loadResultsSuccessEvent"/*event name*/, 
								this.handleSearchResultsLoaded/*event listener*/);
			this.context.listen(this, "loadResultsErrorEvent"/*event name*/, 
								this.handleSearchResultsLoadError/*event listener*/);
		},		
		close : function() {
			//when view closes, unbind Model bindings
			this._modelBinder.unbind();
		},
		onRender : function() {
			
			this._modelBinder.bind(this.model/*the model to bind*/, 
								   this.el/*root element*/, 
								   this.bindings /*bindings*/ );
								   
						   
		},
		handleSearchResultsLoaded : function(data){
			this.model.clear();
			this.model.set(data);
		},
		handleSearchResultsLoadError : function(){
			this.model.clear();
			alert('Opps...something went wrong, try searching again');
		}
	});

	return ResultView;

})

```
### Container View

```javascript
//container view
define([
	"jquery",
	"underscore", 
	"backbone", 
	"marionette",
	"geppetto", 
	"src/controller/ApplicationContext", 
	"text!src/templates/ContainerTemplate.html", 
	"src/views/SearchView", 
	"src/views/ResultView"],
	function(
		$, 
		_, 
		Backbone, 
		Marionette,
		Geppetto, 
		ApplicationContext, 
		ContainerTemplate,
		SearchView, 
		ResultView) {

	//container view acts as plain layout view
	var ContainerView = Marionette.ItemView.extend({
		//set template
		template : ContainerTemplate,
		className : "container myContainer",
		initialize : function() {
			 _.bindAll(this);
			//create a Geppetto context
			Geppetto.bindContext({
				view : this,
				context : ApplicationContext
			});
		},
		onRender : function() {
			//when view is container view is rendered
			//construct the search view
			this.constructSearchView();
			//construct the result view
			this.constructResultView();
		},
		constructSearchView : function() {
			//instantiate an search view
			//notice that we are passig the context from the
			//current container view to the search view constructor
			var mySearchView = new SearchView({
				context : this.context,
				model : new Backbone.Model()
			});
			//render the view
			mySearchView.render();
			//append it the current container
		this.$el.append(mySearchView.$el);

		},
		constructResultView : function() {
			//instantiate an result view
			//notice that we are passig the context from the
			//current container view to the result view constructor
			var myResultView = new ResultView({
				context : this.context,
				model : new Backbone.Model()
			});
			//render the view
            myResultView.render()
			//append it the current container
			this.$el.append(myResultView.$el);
			

		}
	});
	return ContainerView;
});
```

# So, who is actually fetching the data ?

By looking at the code above, none of the views hold the business logic to fetch the data from the serve. The `search view` simply dispatches a `performSearchEvent` with the movie title as payload. The `result view` keeps listening for `loadResultsSuccessEvent` or `loadResultsErrorEvent` for displaying the data or error message & the `container view` simply creates these two views.

So who is actually querying the server ? Well, with `Geppetto`, you could define commands that lets you handle all the complex business logic.

The code snippet below shows the command for our example. The commands have an `execute` function which gets called when an event tied to the command is triggered.


```javascript
//Search Movies Command
define(["jquery", "underscore"], function($, _) {


	var command = function() {
	};

	command.prototype.execute = function() {
		_.bindAll(this);
		var that = this;

		var apikey = "78ejsdd76tc6jsffmrxjddxu";
		var baseUrl = "http://api.rottentomatoes.com/api/public/v1.0";
		var moviesSearchUrl = baseUrl + '/movies.json?apikey=' + apikey;
		//get the movie title
		var query = this.eventData.data.get("title");
		var pageLimit = "&page_limit=1";

		//make an plain jquery ajax call to fetch the movie details using the
		//rotten tomatoes public api's

		$.ajax({
			url : moviesSearchUrl + '&q=' + encodeURI(query) + pageLimit,
			dataType : "jsonp",
			success : function(data) {
				that.handleDataLoadSuccess(data);
			},
			statusCode : {
				503 : function() {
					that.handleDataLoadError("page not found");
				}
			},
			error : function(jqXHR, textStatus, errorThrown) {
				that.handleDataLoadError(errorThrown);
			}
		});

	};

	command.prototype.handleDataLoadSuccess = function(data) {
		var movies = data.movies;

		if (!data || !data.movies || data.movies.length <= 0) {
			//when there are no movies dispatch an error event
			this.context.dispatch("loadResultsErrorEvent"/*event name*/);
		} else {
			//when we get the movies results
			//construct an object with movie details
			var resultObj = {};
			resultObj.rated = movies[0].mpaa_rating;
			resultObj.title = movies[0].title;
			resultObj.rating = movies[0].ratings.audience_score;
			resultObj.year = movies[0].year;
			resultObj.poster = movies[0].posters.original;
			//dispatch an event on the context with movie details as payload
			this.context.dispatch("loadResultsSuccessEvent"/*event name*/, resultObj);
		}

	};

	command.prototype.handleDataLoadError = function(e) {
		//when there are no movies dispatch an error event
		this.context.dispatch("loadResultsErrorEvent"/*event name*/);
	};

	return command;


})

```

# How does this command get called?

`Geppetto` controller/context facilitate's the mappings between events and command. In our case whenever a `performSearchEvent` is dispatch the context/controller maps it to the `SearchMoviesCommand` and supplies the `eventData` as payload to the command.

```javascript
//application context or controller
define([
	'backbone', 
	'geppetto',
	'src/commands/SearchMoviesCommand'], 
function(
	Backbone,
	Geppetto, 
	SearchMoviesCommand) {

	//return a geppetto context
	return Geppetto.Context.extend({
		//setup an initialize function
		initialize : function() {
			// map commands 
			//when ever a "performSearchEvent" is dispatch on this command
			//the context delegates that call to the SearchMoviesCommand
			this.mapCommand( "performSearchEvent"/*event name*/, SearchMoviesCommand );
		}
	});
})

```

If we look at the initialize function in container view, we create a context using the bindContext function.

```javascript
...
Geppetto.bindContext({
	view : this,
	context : ApplicationContext
});
...
```

We then pass this context into search and result view constructors. Using context, communicating between view becomes really easy. Each view that shares a common context can dispatch and listen to events on the context.

# So what's the benefit of Geppetto

By now, its should be clear as to what advantage Geppetto brings to your Backbone apps. Here is a list of them:

* Currently we use the rotten tomatoes api's for searching movies. If we want to use a different service provider, all we need to do is modify the logic in the command. Our views remain untouched
* The same logic goes for the views as well, if we want to change the view layout, the business logic remains untouched.
* Since there is a clear separation of concern and de-coupling between our views and command, writing test cases would be really easy.

# Where can I find the complete source code for this example?

The complete source code for this example can be found at the following repo [link](https://github.com/niki4810/Developing-Modular-Apps-With-Geppetto)

Here is a direct link for the [zip file](https://github.com/niki4810/Developing-Modular-Apps-With-Geppetto/archive/master.zip)

# Credits

* [Geppetto's](http://modeln.github.io/backbone.geppetto/) project page has detailed documentation on all of its features, please refer to it for further details. 
* Special thanks to [David Cadwallader](https://github.com/geekdave) for building such a elegant framework.
* The example make uses of the [rotten tomatoes developer api](http://developer.rottentomatoes.com/) for fetching the movie details.



 


 