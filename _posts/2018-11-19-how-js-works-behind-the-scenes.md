---
title: Unveiling the Curtain | How Javascript works behind the scenes?
---

So you clicked run or you refreshed your browser, what happens?


First, code gets read line by line by a parser, checking for syntax errors first. If it encounters an error, instead of finishing the entire code, it quickly throws an error and stops the execution. If it does not encounters any errors, parser produces a data structure known as Abstract Syntax Tree-

In computer science, an abstract syntax tree (AST), or just syntax tree, is a tree representation of the abstract syntactic structure of source code written in a programming language. Each node of the tree denotes a construct occurring in the source code.

Then, our javascript code gets translated into machine code. This is the general idea.


To be able to run a javascript code, we need an environment. Environments which are required to run javascript are usually called Execution Contexts. One can think of execution context as a wrapper in which our code gets evaluated and executed. The default execution context is called the Global Execution Context, which is usually the windows object in browsers. In javascript, the code that is NOT in any function, belongs to the global object. So, declaring a variable called A is the same thing as window.A as shown below.

{% highlight javascript linenos %}
A === window.A;
{% endhighlight %}

Each time we write a function in the global execution context, a new execution context is added to the execution stack. So, if one creates a code like this;

{% highlight javascript linenos %}
var a=3;

function() {
	b=a+2;
}
{% endhighlight %}

The objects and the execution stack would look like this.

Picture


Let's dive a little bit deeper into execution context.

Javascript engine creates the execution context in the following 2 stages, first being creation phase and second being execution phase. One can think of these stages as properties of execution context.

1-Creation Phase

There are 3 major steps that happens in the creation phase. Firstly, an object called Variable Object gets created. In this step, code is scanned for function and variable declarations. For each variable or function, a property is created in the Varible Object and for variables, it is set to undefined. This process is also referred to as "hoisting". This also means that functions and variables are availabe before execution of the code in javascript. This means that the code below,

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

Picture

For the first line, we get an 'undefined'. This is because, variables are set to 'undefined' in the creation phase. Third line is processed as normal and variable 'a' is logged as 1. 

Generally speaking, the most important use case for hoisting is not variables but function declarations.


Second step in the creation phase is creation of Scope Chain. This step is the answer of "Where can we access a certain variable or function?" In javascript, each function creates a scope, which is an environment in which the variables that it defines are accessible. Also, each function can access to its parent functions scope. What do I mean by this? Let's check the code below.

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

Picture

Lexically speaking, variable 'c' is declared in function second's scope, variable 'b' is declared in function first's scope and variable 'a' is declared in the global scope. Since the console.log() function is called in function second, it has access to its parent function scope which is function first and it has access to global scope therefore, making it able to access all 3 variables, a, b and c. However, this does NOT work backwards.

Picture

Hence, the name, Scope Chain.

The third and the last step in the creation phase is determining the value of 'this' keyword. As the name itself suggests, 'this' keyword refers to the object it belongs to.

In a regular function call and in general 'this' keyword  point at the global object which is the window object in the browser. However in a method call, 'this' variable points to the object that is calling the method WHEN it is called. One will have a better understanding of this concept when the code below is evaluated.

{% highlight javascript linenos %}
<script>

console.log(this);

var a={
	name: 'a',
    value: 1,
    foo: function() {
    	console.log(this);
    }
}

a.foo();

</script>
{% endhighlight %}

First usage of 'this' keyword does not belong to any method, therefore, it will point to the Window object, second usage of 'this' keyword belongs to the object 'a' so the log function will log its attributes to the console. Output of the code is shown below.

Picture


2-Execution Phase

In this stage, assignments to all variables and functions are done and the code gets converted into machine executable instructions. But how?

It is important to know that Javascript is single threaded, that means only one statement is executed at a time. As the javascript engine processes the code line by line, it uses the execution stack to keep track of codes that are supposed to run in their respective order. 
