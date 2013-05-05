---
layout: post
title: "Databinding using Backbone Stickit"
date: 2013-05-04 10:56
comments: true
categories: Backbone Backbone-Stickit Data-Binding
---

In my previous [post](http://niki4810.github.io/blog/2013/03/02/new-post/) we looked at setting up data-bindings between our backbone views and models using the `Backbone.Modelbinder` plugin. In this post, I will demonstrate another viable alternative for data-binding called `Backbone.Stickit`.<!-- more -->  

# How can we achieve data-binding using `Backbone.Stickit`?

To illustrate this we will build the same application as we did in the previous [post](http://niki4810.github.io/blog/2013/03/02/new-post/). The only difference is that the example in this post uses `Backbone.Stickit` for `data-binding`.

* The annotated source code for this example can be found at the following [link](http://niki4810.github.io/annotate-sources/databinding-using-stickit.html)
* The complete fiddle can be found at the following [link](http://jsfiddle.net/niki4810/yQjPD/)
* A full screen preview of this application can be found at the following [link](http://jsfiddle.net/niki4810/yQjPD/embedded/result/)

Surprisingly, the changes I had to make in order to convert my previous example to use Backbone.Stickit were pretty minimal.  To start off the bindings maps for the `editor` and the `preview` views are as follows.

```javascript
var editorViewBindings = {
		'[name = "firstName"]' : "firstName", 
		'[name = "lastName"]' : "lastName",
		'[name = "salary"]':{
			observe: 'salary',
			onGet: 'salaryConverter'
			} ,
		'[name = "pro"]' : "pro",
		'[name = "favSearch"]': "favSearch"
		};
		
var viewerBindings = {
					'[name = "firstName"]' : "firstName", 
					'[name = "lastName"]' : "lastName",
					'[name = "salary"]':{
						observe: 'salary',
						onGet: 'salaryConverter'    
						} ,
					'[name = "pro"]' : "pro",
					'[name = "favSearch"]' : {
						observe : 'favSearch',
						update: function($el, val, model, options) {
							$el.text(val);
							$el.attr("href",val);
							}
						}
					}

```

Some notable differences are that, the bindings map declarations are opposite to that of ModelBinder. Here we specify the selector as a `key` on the bindings map and the value is the attribute name on the `model`. For example, the element with `name` property set to `firstName` will be bound to the `firstName` property on the model.

### using the `onGet` callback function

As we might recollect, we have a requirement in our application to format the salary field when it gets displayed. To achieve this through `Stickit` we make use of the `onGet` callback in the bindings. As we can see from the code below, the bindings for the salary field looks a little different from others. Its declared as an object with two properties `observe` and `onGet`

```javascript

var editorView = {
	...
		'[name = "salary"]':{
			observe: 'salary',
			onGet: 'salaryConverter'
			} ,
	...
}

```

According to stickit documentation, the `observe` property is a string or an array which is used to map a model attribute to a view element. The `onGet` is a  callback which returns a formatted version of the model attribute value that is passed in before setting it in the bound view element. The `salaryConverter` will be a function that is defined with in our backbone view.


### using the `update` callback function

In order to achieve our next requirement, i.e when we change our fav search engine on the editor view, we should change the label and the href property on the anchor element in the preview view. To achieve this we make use of the `update` callback function with in our model. The code below shows the bindings

```javascript
var viewerBindings = {
					...
					'[name = "favSearch"]' : {
						observe : 'favSearch',
						update: function($el, val, model, options) {
							$el.text(val);
							$el.attr("href",val);
							}
						}
					}

```

According to the Stickit documentation `update` is a callback which overrides stickit's default handling for updating the value of a bound view element. The callback function gives a handle to the view's bounded element. Using this we set the text and the href property on the model.


Finally, with in our view render function we call `stickit` and pass in the model and bindings.

```javascript

//create  a Backbone view
	var BaseView = Backbone.View.extend({
		close: function () {
			//when view closes, call unstickit to unbind Model bindings
			this.unstickit();
		},
		render: function () {
			//when the view is rendered
			//get the templated id from passed in options
			//NOTE: templateId is not a property of Backbone or Backbone Stickit, its a custom parameter that we pass into view's constructor           
			var templateId = "#" + this.options.templateId;
			//construct the template
			var template = _.template($(templateId).html());
			var templateHTML = template();
			//append it to current view
			this.$el.html(templateHTML);
			this.stickit(this.model,this.options.bindings);
			return this;
		},
		//create a converter function, that formats the 
		//given value as money, for example 123 gets converted to
		//$123.00, used by the money input.
		salaryConverter : function(value,options){
			return  accounting.formatMoney(value);
		}
	});
```

# Stickit vs ModelBinder, which one should I use?

In my opinion, both of these are awesome frameworks for data-binding. As we can see form the application we've built, both these frameworks provide a great way for binding your models to views. 

I did not investigate much in terms of which is efficient in terms of performance so I cannot really comment on either of these frameworks in terms of their performance. 

So, if you are planning to implement data-binding for your own application, choose a framework which best meets your application requirements and architecture.
 

# About Backbone.Stickit

More detail's about Stickit it  can be found on their [project page](http://nytimes.github.io/backbone.stickit/). Special thanks to [Matthew DeLambo](https://github.com/delambo) for developing yet another awesome data-binding framework for Backbone.


