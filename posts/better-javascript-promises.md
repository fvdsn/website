---
title: Better Javascript Promises
description: Old article that tried to make a good promise system before async await, still relevant I think.
date: 2014-04-25
layout: layouts/post.njk
hn: https://news.ycombinator.com/item?id=7647116
---

> This is a repost of an old article from 2014 that was lost in a blog migration but I think is still
> relevant. It shows how differently the javascript promise problem could have been solved. This was written
> before async await became a thing, and before function coloring problem was given its name. This proposal
> would have solved the promise problem without the coloring, but alas, it wasn't liked then.
>
> The post makes mention of the concept of `Deferred` this was the popular name for the `Promise` class at the time,
> and without `async` and `await`, we had to manually use `.then()` and `.pipe()` in our javascript code. This was a major
> pain in the ass and the motivation behind the post. But this posts explores a completely different solution using dataflow programming, a feature of the Oz programming language that I really liked at the time.
>
> Good reading !

If you ever had to port localStorage based code to IndexedDB you have been confronted to Javascript's biggest flaw; its asynchronous system is leaky. Once you have an asynchronous method, every other method that calls it must be asynchronous as well.

You cannot escape asynchronous code, once it's there it will contaminate the rest of your system slowly turning everything into callbacks.

There is no current solution to this problem. If you try replace a callback with a Deferred, you end up with a deferred and another callback. Deferreds don't replace callbacks, they move them around.

Which is better than nothing and that probably explains their success. Still, they are complex beasts. Even after extensive use most programmers still have a hard time dealing with them. I have seen entire articles exploring the best ways to make three parallel ajax calls on the frontpage of HackerNews. This is not the sign of a good state of affairs.

It is unfortunately not possible to fix this without making changes to Javascript. The fix I propose is a simple addition to javascript, inspired from promises and Oz's dataflow programming. The Javascript syntax is left unchanged and only one new keywords is added. But most importantly it makes real world code shorter and more readable, making many common asynchronous programming patterns trivialy easy.

## The future entity

```js
var a = future;
```
The future entity indicates that the actual value or reference of the variable is not yet available. When an expression tries to evaluate future the code pauses and returns the hand to the scheduler. When we later assign a value or reference to that variable, the waiting code resumes.

```js
var a = future;
setTimeout(function(){
    a = 'Hello World';
},1000);
console.log(a); // waits 1sec and then prints 'Hello World'
```

Assigning future to another variable or passing it as a function parameter does not pause the execution. So if we reimplement `debug.log()` properly, the following example is equivalent to the previous.

```js
var a = future;
console.log(a); // prints an empty line that will show 'Hello World' in 1 second
setTimeout(function(){
    a = 'Hello World';
},1000);
```

We call resolution the act of assigning a value or reference to a variable containing future. Once a future is resolved, it becomes the assigned value. And so do all the other variables linked to the same future. This is what allows the paused expressions to resume.

A variable containing a future is special only for as long as it contains a future. After resolution no trace of the future is left and it behaves just like any other variable. This is one of the key differences between future and Deferreds

```js
var a = future;
console.log(a);     // will print 'hello'
a = 'Hello'
a += ' World!'
console.log(a);     // will print 'Hello World!'
```
## An ajax use case.

We could implement an ajax library that returns futures. Those would later be resolved by the network responses. The next example makes use of that hypothetical implementation.

```js
function renderCard( employeeId ){
    var employee = ajax('/employee/'+employeeId);
    var picture  = ajax('/pictures/employee/'+employeeId);
    return '<h1>'+employee.name+'</h1><img src="'+picture+'"/>';
}
```
In this example, the ajax calls immediately returns a future. The two ajax calls will execute in parallel. The string concatenation will then block when evaluating the futures. This future example is massively simpler than the Deferred or callback equivalent.

The first great thing is that future let us code asynchronously as we would do synchronously. The code is easier to write, and easier to understand.

The second great thing is that the asynchronous ajax calls are completely hidden inside `renderCard()`. The fact that we used ajax calls to implement the function is an implementation detail. It does not impact the signature of the function. This would not have been the case had we used callbacks or Deferreds.

Calling `renderCard()` will block the caller for the duration of the two ajax calls, and this may hurt the performances. The fix is easy: We wrap the function code in a setTimeout and the return value is wrapped in a future.

```js
function renderCard( employeeId ){
    var html = future;
    setTimeout(function(){
        var employee = ajax('/employee/'+employeeId);
        var company  = ajax('/company/'+employee.companyId);
        html = '<h1>'+employee.name+'</h1><h2>'+company.name+'</h2>';
    });
    return html;
}
```

`renderCard()` now immediately returns a future that will be resolved on its own once the requests and string operations have completed.

It is now completely asynchronous, and we haven't changed the function signature. Returning future instead of a string does not impact the calling code as it will become a string once evaluated.

And the following example shows another huge advantage of futures over Deferreds: we can mix synchronous and asynchronous code:

```js
var html;
if (employeeId === 0) {
    html = '<h1>Administrator</h1>';
} else {
    html = renderCard(employeeId);
}
target.innerHtml = html;
```

## Future Functions

Declaring a function that executes asynchronously and wraps the return value with a future will be a common pattern. We thus introduce the following syntaxic sugar; prepend future to a function declaration and it will always be executed asynchronously.

```js
future function renderCard( employeeId ){
    var employee = ajax('/employee/'+employeeId);
    var company  = ajax('/company/'+employee.companyId);
    return '<h1>'+employee.name+'</h1><h2>'+company.name+'</h2>';
}
```

future functions could also be used as another syntax for setTimeout(...,0) with the added benefit of being safe to use inside loops.

The following example shows how to fetch and process multiple asynchronous resources in parallel

```js
results = [];
for (var i = 0; i < list.length; i++){
    (future function(i){
        results.push(process(ajax('/foo/'+list[i])));
    })(i);
}
```

## Assigning futures

futures are neither values nor references. Their behaviour on evaluation and asignment obey different rules. When they are assigned values or reference, they resolve. When they are evaluated, the code pauses until they are resolved. But what happens when we assign a future to another variable ?

When we assign a future to a variable, a new future is created. That future is a made a child of the assigned future. When a future is resolved, all its child future are resolved. But resolving a child future does not resolve its parent. This behaviour is easier to understand by seeing it in practice.

```js
var a = future;
var b = a;          // b is the child future of a
console.log(a,b);   // will print 'hello' 'hello'
a = 'hello';

var a = future;
var b = a;
console.log(a,b);   // will print 'hello' 'world'
b = 'world';        // this does not resolves 'a'
a = 'hello';
```

You'll note that once a child future is resolved, it is not a future anymore. It's resolved value is not affected if the parent is later resolved.

## Resolution Scoping

A future is thus tied to it's original variable. It can only be resolved by assigning a value to the original variable. The resolution is thus scoped to the function where the variable was declared.

To use a Deferred analogy, only Promises can be returned or passed as parameters.

This is a very useful behaviour as it prevents the asynchronous calls made inside a function from being tampered from the outside.

## Waiting

Sometimes you'll want to make sure a future has resolved, which is exactly what the window.wait() function would do. It takes one or multiple parameters and waits for their resolution before returning the resulting value (or a list of them).

```js
var foo = ajax('/foo');
var bar = ajax('/bar');
wait(foo,bar);

console.log('Done!');
```

We can now revist the previous example of parallel resource loading and add a synchronisation step where we wait for the processing:

```js
var results = [];
var jobs = [];

for (var i = 0; i < list.length; i++){
    jobs.push((future function(i){
        results.push(process(ajax('/foo/'+list[i])));
        // returns undefined and resolves the job.
    })(i));
}

wait.apply(window,jobs);
console.log('Done!', results);
```

## Exceptions

We can use Exceptions to indicate the failure to resolve a future. When an exception propagates trough a context defining a future, or when it is uncaught, we can assume the related futures will not be correctly resolved. All the unresolved futures thus resolve to a throw of that Exception.

```js
future function renderCard( employeeId ){
    try {
        var employee = ajax('/employee/'+employeeId);
        var company  = ajax('/company/'+employee.companyId);
        return '<h1>'+employee.name+'</h1><h2>'+company.name+'</h2>';
    } catch ( error ) {
        return '<section> Error:'+error.errno+'</section>';
    }
}
```

## Yield & Generators

When writing this article I was not aware that generators could eliminate callbacks and solve the very problem I tried to solve in this proposal. [This excellent article explains how to do it](https://web.archive.org/web/20140412145738/http://jlongster.com/A-Study-on-Solving-Callbacks-with-JavaScript-Generators).

One one hand, I am extremely glad that we have now a solution to callback hell and asynchronous code encapsulation. What's more, the yield based solution looks extremely close in syntax and features to the futures we have been talking about.

Unfortunately, using yield based solution will require third party libraries, which means increased payload and incompatibilities among frameworks.

And I am also quite disappointed that the solution to one of JavaScripts biggest flaws requires hacking a language feature that was designed for an entirely different use-case, which will surely bring lots of confusion.

I think JavaScript deserves a native and straightforward way to handle asynchronous code beyond callbacks.
