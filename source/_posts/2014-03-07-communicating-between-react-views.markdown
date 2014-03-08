---
layout: post
title: "Communicating Between React Views"
date: 2014-03-07 20:40
comments: true
categories: Facebook React, React JS, Databinding, Communicating
---

In my hunt to learn the next UI framework after Backbone, I came across [React](http://facebook.github.io/react/index.html). Its an open source javascript famework from Facebook, used for building modular user interfaces. As the project page says, most people use React as `V` in the `MVC` architecture.<!-- more -->

While the React project page has a ton of documentation and examples of what it is and what it can do, I could not find a good example on how two React views can communicate with each other.

In this blog post, I would like to share my experience in building a simple example that looks like [this](http://jsfiddle.net/niki4810/t8pDk/show/result/). 

The application contains three views, `DisplayView`, `EditorView` (stateless) and the `ContainerView` (stateful). When we type something in the `Editorview` we see that the `DisplayView` gets updated automatically.

# Step 1 : Constructing the DisplayView

The code for the Display view looks like this

```javascript

/** @jsx React.DOM */

var DisplayView = React.createClass({
    render : function(){
        return (
            <div>
                <span>Name :</span>
                <span>{this.props.text}</span>
            </div>
        );
    }
});

```
We simply create a React view with a render function that returns a template. The interesting point to note here is the `{this.props.text}` in the second span. 

If we want to render this view as is, and append it to the body, we can simple use

```javascript
	//just a sample initialization, we will not be rendering DisplayView this way in our example
	React.renderComponent(<DisplayView text="Bob"/>, document.body);
```

The above call with render the DisplayView and set the text in the second span as `Bob`. As you might have guessed, `this.props` refers to all the props that you send in the `<DisplayView />` tag. The props could be string variables, or a callback functions.




# Step2: Constructing the EditorView

The code for the Editor view looks like this

```javascript

var EditorView = React.createClass({
    render : function(){
        return (
            <div>
                <span>
                    Name :
                 </span>
                <span>
                    <input type="text" 
                       onChange={this.props.onChange}
                       placeholder="type your name here..."/>
                 </span>
            </div>
        );
    }
});
	
```
Similar to the `DisplayView`, we the render function returns a template. The template contains a label and a input component. The input has a onChange event listener which set to `this.props.onChange'. Which means that if we wanted to use the EditorView we would need to pass it as a prop. The call would look something like this.

```javascript
	//just a sample initialization, we will not be rendering EditorView this way in our example
	var foo = function(e){
		console.log(e.target.value); //returns the value of the text input
	}
	React.renderComponent(<EditorView onChange={foo}/>, document.body);
```

# Step3 : Constructing the ContainerView

The code for ContainerView looks like this 

```javascript
	var ContainerView = React.createClass({
    getInitialState : function(){
        return {text: ""}
    },
    handleChange : function(e){
        var currentText = e.target.value;
        this.setState({text : currentText});
    },
    render : function(){
        var text = this.state.text;
        return (  
                <div>
                    <EditorView onChange={this.handleChange}/>
                    <DisplayView text={text}/>
                </div>    
        );
    }
});
 
//This will render the ContainerView and append it to the body 
React.renderComponent(<ContainerView />, document.body);
```

Based on the React's documentation on [states](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html#how-state-works). A common patter that React suggests is to have multiple `stateless` views which take `props` from a single `stateful` view and re-render themselves.

- Both the `EditorView` and the `DisplayView` are state less, while the the `ContinerView` is stateful. 
- The `getInitialState` returns an initial state object with a text property. 
- The `render` function reads this text property from the state and supplies it to the DisplayView and sets a `onChange` handler on the EditorView.
- And finally the `handleChange` is the callback function for the `EditorView`'s `input` onChange event. 

Everytime `this.setState` is called, it leads to a call to the `render` function there by re-rendering all the sub views with in the ContainerView. This way, both the EditorView and the DisplayView are able to communicate with each other.

# Does re-rendering cause a performance issue ?

React claims that this re-rendering is not expensive as it does not go against the tradational DOM, but rather an in memory virtual DOM, which does a diff on what's changed from the previous state and re-renders only those portions. Thus, giving a significant performance boost for your views. This is one of the selling points for React.


Finally, the complete source code for this example can be found at the following [fiddle](http://jsfiddle.net/niki4810/t8pDk/). Hope this example helps you understand how your React views can talk to each other.









