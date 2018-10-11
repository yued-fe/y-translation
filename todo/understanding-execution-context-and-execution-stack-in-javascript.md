> * 原文地址：[Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)

> * 译文地址：[理解Javascript执行上下文和执行栈](https://github.com/yued-fe/y-translation/blob/master/todo/understanding-execution-context-and-execution-stack-in-javascript.md)

> * 译者：[夏佳昊](https://github.com/TJ-XiaJiaHao)

> * 校对者：[盛新原](https://github.com/shengxinyuan)、 [谈伟](https://github.com/Tvinsh)、
[时磊](https://github.com/nervouself)、
[张成荣](https://github.com/smadey)

Understanding Execution Context and Execution Stack in Javascript
=================================================================

If you are or want to be a JavaScript developer, then you must know how the JavaScript programs are executed internally. The understanding of execution context and execution stack is vital in order to understand other JavaScript concepts such as Hoisting, Scope, and Closures.

Properly understanding the concept of execution context and execution stack will make you a much better JavaScript developer.

So without further ado, let’s get started :)

#### Easily share JS code across projects with Bit

Bit’s [open source platform](https://github.com/teambit/bit) helps you easily share and manage reusbale code code across projects, and sync changes from anywhere. Any team member can discover, use and develop shared code, suggest updates and stay in sync.

### What is an Execution Context?

Simply put, an execution context is an abstract concept of an environment where the Javascript code is evaluated and executed. Whenever any code is run in JavaScript, it’s run inside an execution context.

#### Types of Execution Context

There are three types of execution context in JavaScript.

*   **Global Execution Context —** This is the default or base execution context. The code that is not inside any function is in the global execution context. It performs two things: it creates a global object which is a window object (in case of browsers) and sets the value of `this` to equal to the global object. There can only be one global execution context in a program.
*   **Functional Execution Context —** Every time a function is invoked, a brand new execution context is created for that function. Each function has its own execution context, but it’s created when the function is invoked or called. There can be any number of function execution contexts. Whenever a new execution context is created, it goes through a series of steps in a defined order which I will discuss later in this article.
*   **Eval Function Execution Context —** Code executed inside an `eval` function also gets its own execution context, but as `eval` isn’t usually used by JavaScript developers, so I will not discuss it here.

### Execution Stack

Execution stack, also known as “calling stack” in other programming languages, is a stack with a LIFO (Last in, First out) structure, which is used to store all the execution context created during the code execution.

When the JavaScript engine first encounters your script, it creates a global execution context and pushes it to the current execution stack. Whenever the engine finds a function invocation, it creates a new execution context for that function and pushes it to the top of the stack.

The engine executes the function whose execution context is at the top of the stack. When this function completes, its execution stack is popped off from the stack, and the control reaches to the context below it in the current stack.

Let’s understand this with a code example below:

```
let a = 'Hello World!';

function first() {  
  console.log('Inside first function');  
  second();  
  console.log('Again inside first function');  
}

function second() {  
  console.log('Inside second function');  
}

first();  
console.log('Inside Global Execution Context');
```

![](https://cdn-images-1.medium.com/max/2000/1*ACtBy8CIepVTOSYcVwZ34Q.png)

An Execution Context Stack for the above code.

When the above code loads in the browser, the Javascript engine creates a global execution context and pushes it to the current execution stack. When a call to `first()` is encountered, the Javascript engines creates a new execution context for that function and pushes it to the top of the current execution stack.

When `second()` function is called from within `first()` function, the Javascript engine creates a new execution context for that function and pushes it to the top of the current execution stack. When`second()` function finishes, its execution context is popped off from the current stack, and the control reaches to the execution context below it, that is `first()` function execution context.

When the `first()` finishes, its execution stack is removed from the stack and control reaches to the global execution context. Once all the code is executed, the JavaScript engine removes the global execution context from the current stack.

### How is the Execution Context created?

Up until now, we have seen how the JavaScript engine manages the execution context, Now let’s understand how an execution context is created by the JavaScript engine.

The execution context is created in two phases: **1) Creation Phase** and **2) Execution Phase.**

### The Creation Phase

Before any JavaScript code is executed, the execution context goes through the creation phase. Three things happen during the creation phase:

1.  Value of **this** is determined, also known as **This Binding**.
2.  **LexicalEnvironment** component is created.
3.  **VariableEnvironment** component is created.

So the execution context can be conceptually represented as follows:

```
ExecutionContext = {  
  ThisBinding = <this value>,  
  LexicalEnvironment = { ... },  
  VariableEnvironment = { ... },  
}
```

#### **This Binding:**

In the global execution context, the value of `this` refers to the global object. (in browsers, `this` refers to the Window Object).

In the function execution context, the value of `this` depends on how the function is called. If it is called by an object reference, then the value of `this` is set to that object, otherwise the value of `this` is set to the global object or `undefined`(in strict mode). For example:

```javascript
let person = {  
  name: 'peter',  
  birthYear: 1994,  
  calcAge: function() {  
    console.log(2018 - this.birthYear);  
  }  
}

person.calcAge();   
// 'this' refers to 'person', because 'calcAge' was called with //'person' object reference

let calculateAge = person.calcAge;  
calculateAge();  
// 'this' refers to the global window object, because no object reference was given
```

#### Lexical Environment

The [official ES6](http://ecma-international.org/ecma-262/6.0/) docs defines Lexical Environment as

> A _Lexical Environment_ is a specification type used to define the association of _Identifiers_ to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an _outer_ Lexical Environment.

Simply put, A _lexical environment_ is a structure that holds **identifier-variable mapping**. (here **identifier** refers to the name of variables/functions, and **variable** is the reference to actual object \[including function type object\] or primitive value).

Now, within the Lexical Environment, there are two components: (1) the **environment record** and (2) a **reference to the outer environment**.

1.  The **environment record** is the actual place where the variable and function declarations are stored.
2.  The **reference to the outer environment** means it has access to its outer lexical environment.

There are two types of _lexical environment_:

*   A **global environment** (in a global execution context) is a Lexical Environment which does not have an outer environment. The global environment’s outer environment reference is **null**. It has global object (window object) and its associated methods and properties (eg. array methods) inside the environment record as well as any user-defined global variables, and the value of `this` refers to the global object.
*   A **function environment**, in which the user-defined variables inside the function are stored in the **environment record**. And the reference to the outer environment can be the global environment, or an outer function environment that contains the inner function.

**Note —** For **function environment**, the _environment record_ also contains an `arguments` object that contains mapping between indexes and arguments passed to the function and the _length(number)_ of the arguments passed into the function. For example, an argument object for the below function looks like this:

```
function foo(a, b) {  
  var c = a + b;  
}  
foo(2, 3);

// argument object  
Arguments: {0: 2, 1: 3, length: 2},
```

There are also two types of **_environment record_**  (see above!):

*   **Declarative environment record** stores variables, functions, and parameters. A function environment contains declarative environment record.
*   **Object environment record** is used to define association of variables and functions appeared in the _global execution context._ A global environment contains object environment record.

Abstractly, the lexical environment looks like this in pseudocode:

```
GlobalExectionContext = {  
  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      _// Identifier bindings go here_ }  
    outer: <null>  
  }  
}

FunctionExectionContext = {  
  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      _// Identifier bindings go here_ }  
    outer: <Global or outer function environment reference>  
  }  
}
```

#### Variable Environment:

It’s also a Lexical Environment whose EnvironmentRecord holds bindings created by _VariableStatements_ within this execution context.

As written above, the variable environment is also a lexical environment, So it has all the properties of a lexical environment as defined above.

In ES6, one difference between **LexicalEnvironment** component and the **VariableEnvironment** component is that the former is used to store function declaration and variable (`let` and `const`) bindings, while the latter is used to store only variable `(var)` bindings.

Let’s look at some code example to understand the above concepts:

```
let a = 20;  
const b = 30;  
var c;

function multiply(e, f) {  
 var g = 20;  
 return e * f * g;  
}

c = multiply(20, 30);
```

The execution context will look something like this:

```
GlobalExectionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      // Identifier bindings go here  
      a: < uninitialized >,  
      b: < uninitialized >,  
      multiply: < func >  
    }  
    outer: <null>  
  },

  VariableEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      // Identifier bindings go here  
      c: undefined,  
    }  
    outer: <null>  
  }  
}

FunctionExectionContext = {  
   
  ThisBinding: <Global Object>,

  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      // Identifier bindings go here  
      Arguments: {0: 20, 1: 30, length: 2},  
    },  
    outer: <GlobalLexicalEnvironment>  
  },

VariableEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      // Identifier bindings go here  
      g: undefined  
    },  
    outer: <GlobalLexicalEnvironment>  
  }  
}
```

**Note —** The function execution context will only be created when the call to function `multiply` is encountered.

As you might have noticed that the `let` and `const` defined variables do not have any value associated to them, but `var` defined variables are set to `undefined `.

This is because during the the creation phase, the code is scanned for variable and function declarations, while the function declaration is stored in its entirety in the environment, but the variables are initially set to `undefined` (in case of `var`) or remain uninitialized (in case of `let` and `const`).

This is the reason why you can access `var` defined variables before they are declared (though `undefined`) but get a reference error when accessing `let` and `const` variables before they are declared.

This is, what we call hoisting.

### Execution Phase

This is the simplest part of this entire article. In this phase assignments to all those variables are done, and the code is finally executed.

**Note —** During the execution phase, if the JavaScript engine couldn’t find the value of `let` variable at the actual place it was declared in the source code, then it will assign it the value of `undefined`.

### Conclusion

So we have discussed how JavaScript programs are executed internally. While it’s not necessary that you learn all these concepts to be an awesome JavaScript developer, having decent understanding of the above concepts will help you to understand other concepts such as Hoisting, Scope, and Closures more easily and deeply.

That’s it and if you found this article helpful, please hit the 👏 button and feel free to comment below! I’d be happy to talk 😃

### Shared in Bit’s blog

Bit makes it very easy to share small components and modules between projects and applications, so that you and your team can build faster. Share components, develop them anywhere and create a beautiful collection. Try it.

