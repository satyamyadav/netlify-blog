---
title: understanding es6 pipeline O|>erator
date: "2016-03-02"
categories:
  - code
tags:
  - lodash
  - es6
  - javascript
  - experimental
  - ESNext
---

Pipeline operator `|>` is a readable way of writing chained functions. The operator is currently at stage 1 of tc39 proposal (ESNext Proposal).

<!-- more  -->

The pipeline operator provides a syntactic sugar to call a function with single argument.

```js
expression |> function 

//or call multiple functions as 

expression |> fn1 |> fn2 |> fn3
```

each function gets the result from its preceeding function as input and passes the result to next function i.e. input (argument/expression) is piped through and complete pipeline.

Let's look at some examples :


Define the `double` and `increment` functions to use in examples.

```js
const double = n => n * 2; 

const increment = n => n + 1;
```



**The function(argument) way:**

We can call functions as `g(f(n))` to get our input parameter `n` through the functions `f(n)` and `g(m)`.
_OR by taking intermediate value as a variable:_
`var a = f(n);`
`result = g(a):`

```js
double(increment(double(10)));  //42

// OR

let a = double(10); //20

let b = increment(a); //21

let result = double(b); //42
```



**using lodash**

Using lodash we can achieve the same (chaining) by using `_.chain()` for inbuilt methods or by `_.mixin()` for using custom methods or by `_.flow()` (functional compositions).

```js
const _ = require('lodash');

// using lodash mixin

_.mixin({'double': double, 'increment': increment});

_(10).double().increment().double().value(); //42

// using lodash flow

const doubleIncrementDouble = _.flow([double, increment, double]);

let result = doubleIncrementDouble(10) //42
```


**es5 utility function**

To write a utility function to do the same for us, we need to define a function (`pipe`) that takes a list (array) of methods (functions) and returns a function which takes only one parameter and reduces the functions array from its parent scope initializing with its parameter.  
Now as we have a wrapper function `pipe` we can call it as 
`pipe([fn1, fn2, f3 ...fm])(n)` 
or
 we can define different pipes taking list of functions 
 `var pipeline1 = [double, increment]` 
 and then call it as 
 `pipeline1(a)`.

```js
//  pipeline utility

var pipe = function (functionList) {

  return function (data) {

    return functionList.reduce(function(value, func) {
      return func(value)
    }, data);

  }

}

pipe([double, increment, double])(10); //42
```



**ESNext pipeline operator**
`10 |> double |> increment |> double; ` 

As we often need to transform some data or clean data for some usages like data visualizations , tabular representations, payload generations etc. we need a utility or use a library like lodash for doing that part only, it is good to have feature in language itself as similar to [F#](https://en.wikibooks.org/wiki/F _Sharp_Programming/Higher_Order_Functions#The_.7C.3E_Operator),  [OCaml](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Pervasives.html#VAL%28|%3E%29),  [Elixir](https://www.safaribooksonline.com/library/view/programming-elixir/9781680500530/f_0057.html),  [Elm](https://edmz.org/design/2015/07/29/elm-lang-notes.html),  [Julia](http://docs.julialang.org/en/release-0.4/stdlib/base/?highlight=|%3E#Base.|%3E),  [Hack](https://docs.hhvm.com/hack/operators/pipe-operator), [LiveScript](http://livescript.net/#piping),  as well as UNIX pipes.

Also as in proposal :

...you can use an arrow function to handle multi-argument functions.



--------------------------



`Thank You !!`



>  Your thoughts , suggestions or corrections are most welcome !!, Please feel free to share them below. 
  will test some more complex examples and will share If I found anything interesting.



References :

  [ESNext Proposal tc39](https://github.com/tc39/proposal-pipeline-operator)

  [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Pipeline_operator)

  [Raw code](https://gist.githubusercontent.com/satyamyadav/27723528a35e3d7b4cd80b198ec8f792/raw/e7eb9d3e80ea454d54f17738c0343a9f911d188a/pielinejs.js)



*Gist*

<script src="https://gist.github.com/satyamyadav/27723528a35e3d7b4cd80b198ec8f792.js"></script>


