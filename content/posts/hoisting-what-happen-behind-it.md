+++
categories = ["tech"]
date = "2015-01-01T01:55:22+08:00"
draft = false
slug = ""
tags = ["javascript"]
title = "Hoisting, what happens behind it"

+++

We always see the word **hoisting** in some javascript books and technical blogs. Hoisting is grabbed to explain the phenomenon that we can use a variable before we declare it. That's sound incredible in compiling languages, but in javascript it's really there. Some people might think that the interpretor do a magic to restructure the code and put all variable/function declaration statements at the top of the function body. Thinking like that way might help to write or read the program, but that's not the things truely happen behind. The post is to walk through the mechanism behind hoisting.

<span class="more"></span>

```javascript
function foo(x) {
	console.log(x); // x
	console.log(y); // undefined
	console.log(z); // [function: z]

	z(); // z

	var y = 'y';
	var z = 'z';

	function z() { // overwrite var z
		console.log('z');
	}
}

foo('x');
```
There are 3 kinds of scopes in javascript - `global scope`, `function scope`, `eval scope`. We only discuss function scope here. Whenever a function is invoked, the execution context is put on the top of the stack, and before executing the function code step-by-step, there is a pre-stage called `Declaration Binding Instantiation`. In this declaration binding stage, an `VariableEnvironment`(VE) (in old ECMA-262, it's refered as variable object) is created. Run-time function will refer to this `VariableEnvironment` (actually it should be `LexicalEnvironment` which is initialized the same as `VariableEnvironment`) to identify the variables. So below is what the declaration binding stage do in sequence

1. Formal parameters are bound to VE
2. Function declaration is bound to VE. If the function name is already existing, it will overwrite it.
3. `Arguments` is bound to VE.
4. Variable declaration is bound to VE. If the variable name is already existing, it will do nothing.

VE for `foo` should look like:
```javascript
/*
VE = {
    x: 'x', // formal parameter x
    z: [function z], // reference to function z
    arguments:
    y: undefined
};
*/
```
So before the execution of the function, all variables/functions are bound to VE already to be later referenced. Now you know the magic behind hoisting.

---

http://www.ecma-international.org/ecma-262/5.1/#sec-10.5

