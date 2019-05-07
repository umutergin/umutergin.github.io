---
title: Unveiling the Curtain | How Javascript works behind the scenes?
---

So you clicked run or you refreshed your browser, what happens?


First, code gets read line by line by a parser, checking for syntax errors first. If it encounters an error, instead of finishing the entire code, it quickly throws an error and stops the execution. If it does not encounters any errors, parser produces a data structure known as Abstract Syntax Tree.
>In computer science, an abstract syntax tree (AST), or just syntax tree, is a tree representation of the abstract syntactic structure of source code written in a programming language. Each node of the tree denotes a construct occurring in the source code.

Then, our javascript code gets translated into machine code. This is the general idea.


To be able to run a javascript code, we need an environment. Environments which are required to run javascript are usually called Execution Contexts. One can think of execution context as a wrapper in which our code gets evaluated and executed. The default execution context is called the Global Execution Context, which is usually the windows object in browsers. In javascript, the code that is NOT in any function, belongs to the global object. So, declaring a variable called A is the same thing as window.A as shown below.

{% highlight javascript linenos %}
A === window.A;
{% endhighlight %}

Each time we write a function in the global execution context, a new execution context is added to the execution stack. So, if one creates a code like this;

{% highlight javascript linenos %}
function A(){
        console.log('In the function A execution stack.')
	B();
	console.log('In the function A execution stack.')
}

function B(){
        console.log('In the function B execution stack.')
}
A();
console.log('In the Global Execution Stack')
{% endhighlight %}

The objects and the execution stack would look like this.

![image](/img/execution-context-diagram.png)

First, we have our global context on the execution stack. Then, Function A is called, an execution context that belongs to Function A gets added to execution stack. Then when we are in Function A, Function B gets called, thus, Function B execution context is added to execution stack.
Here is what we see on the browser console when we run this code.

![image](/img/execution-context-code.png)


Let's dive a little bit deeper into execution context.

Javascript engine creates the execution context in the following 2 stages, first being creation phase and second being execution phase. One can think of these stages as properties of execution context.
## 1-Creation Phase
 
There are 3 major steps that happens in the creation phase. Firstly, an object called Variable Object gets created. In this step, code is scanned for function and variable declarations. For each variable or function, a property is created in the Varible Object and for variables, it is set to undefined. This process is also referred to as "**hoisting**". This also means that functions and variables are availabe before execution of the code in javascript. This means that the code below,

{% highlight javascript linenos %}
function pow(a) {
	console.log(a*a);
}

pow(2);
{% endhighlight %}

is the same as this code.

{% highlight javascript linenos %}
pow(2);

function pow(a) {
	console.log(a*a);
}
{% endhighlight %}

In many other programming languages, the second example would throw an error like "pow is not defined" but because of hoisting in javascript, function declaration 'pow' is stored in the variable object before the code is executed. 


However, it is a little bit different with the variables. Let's look at the code below,

{% highlight javascript linenos %}
console.log(a);
var a=1;
console.log(a);
{% endhighlight %}

The output of the code is shown as,

![image](/img/variable-undfined.png)

For the first line, we get an 'undefined'. This is because, variables are set to 'undefined' in the creation phase. Third line is processed as normal and variable 'a' is logged as 1. 

Generally speaking, the most important use case for hoisting is not variables but function declarations.

Second step in the creation phase is creation of **Scope Chain**. This step is the answer of "Where can we access a certain variable or function?" In javascript, each function creates a scope, which is an environment in which the variables that it defines are accessible. Also, each function can access to its parent functions scope. What do I mean by this? Let's check the code below.

{% highlight javascript linenos %}
var a=1;

first();

function first() {
	var b=2;
	second();

	function second() {
		var c=3;
		console.log(a+b+c);
	}
}
{% endhighlight %}

Here is the output of the course.

![image](/img/scopechain1.png)

Lexically speaking, variable 'c' is declared in function second's scope, variable 'b' is declared in function first's scope and variable 'a' is declared in the global scope. Since the console.log() function is called in function second, it has access to its parent function scope which is function first and it has access to global scope therefore, making it able to access all 3 variables, a, b and c. However, this does NOT work backwards.

![image](/img/scopechain2.png)

Hence, the name, Scope Chain.

The third and the last step in the creation phase is determining the value of 'this' keyword. As the name itself suggests, 'this' keyword refers to the object it belongs to.

In a regular function call and in general 'this' keyword  point at the global object which is the window object in the browser. However in a method call, 'this' variable points to the object that is calling the method WHEN it is called. One will have a better understanding of this concept when the code below is evaluated.

{% highlight javascript linenos %}
console.log(this);

var a={
	name: 'a',
        value: 1,
        foo: function() {
    	        console.log(this);
    }
}

a.foo();
{% endhighlight %}

First usage of 'this' keyword does not belong to any method, therefore, it will point to the Window object, second usage of 'this' keyword belongs to the object 'a' so the log function will log its attributes to the console. Output of the code is shown below.

![image](/img/thiskeyword.png)
## 2-Execution Phase
 
In this stage, assignments to all variables and functions are done and the code gets converted into machine executable instructions. But how?

It is important to know that Javascript is single threaded, that means only one statement is executed at a time. As the javascript engine processes the code line by line, it uses the execution stack to keep track of codes that are supposed to run in their respective order. This way of doing one thing before another is called **Synchronous**. In Chrome, every tab is a single Javascript thread and that thread is responsible for every action like scrolling, listening for mouse clicks, etc. When this execution stops, Chrome will throw an error similar to image below.

![image](/img/google-chrome-pages-unresponsive.png)

Since Javascript is a synchronous programming language, it can only do one thing at any given time. Let's say that a program wants to execute a SQL query that pulls lots of records from the database. This request will take a long time, and remember, Javascript is synchronous, does that mean page waits for that SQL request to complete before doing anything else? Well, yes and no. Javascript does one thing at a time, it is true, but there are some solutions for this.

Modern browsers have Web APIs for Javascript to perform operations in the background while continue to execute the rest of Javascript code, allowing Javascrip to become **Asynchronous**. While running javascript code, if a function contains a web api call, that function will be commissioned to the web api with an associated callback function. A callback function is literally derived from calling back. It means once its done with the task, it will call back and deliver the requested content. After entrusting the related function to web api, javascript code continues to get executed normally. Once the execution stack is emptpy, callback functions get pushed into the stack and callback function will be executed, returning the requested content. This process of checking the callback queue to push them into the execution stack if the stack is empty called the **Event Loop**.

Philip Roberts has an awesome [site](http://latentflip.com/loupe/?code=ZnVuY3Rpb24gcHJpbnRIZWxsbygpIHsNCiAgICBjb25zb2xlLmxvZygnSGVsbG8gZnJvbSBiYXonKTsNCn0NCg0KZnVuY3Rpb24gYmF6KCkgew0KICAgIHNldFRpbWVvdXQocHJpbnRIZWxsbywgMzAwMCk7DQp9DQoNCmZ1bmN0aW9uIGJhcigpIHsNCiAgICBiYXooKTsNCn0NCg0KZnVuY3Rpb24gZm9vKCkgew0KICAgIGJhcigpOw0KfQ0KDQpmb28oKTs%3D!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D
) which he explains the event loop. Here is the link below.

