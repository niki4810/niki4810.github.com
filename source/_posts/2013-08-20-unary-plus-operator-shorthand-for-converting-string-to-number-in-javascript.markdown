---
layout: post
title: "Unary plus operator shorthand for converting a string to a number in javascript"
date: 2013-08-20 21:43
comments: true
categories: javascript, hacks
---

If you are a javascript developer like me, you must have come across a scenario where you needed to convert a string to a number. I have generally been using either `parseInt` or `parseFloat` for these type of conversions. <!-- more -->  

For example, you would do something like this for the conversion

```javascript
  var num = parseInt("10"); 
 // output : num = 10;
```
```javascript
   var num = parseFloat("10.23")
  // output: num = 10.23
```

Most recently I have come across another interesting way to do the same conversion, i.e. using the `unary plus` operator ( `+` ). The above two statements can be rewritten as following to achieve the same result 

```javascript
    var num = +"10"
    // output : num = 10;
```
```javascript
   var num =  +"10.23"
  // output: num = 10.23
```

Although choosing one style of conversion might be a personal opinion, I think that converting using the unary plus operator would act as a great shorthand, especially while writing unit test for you javascript code.



