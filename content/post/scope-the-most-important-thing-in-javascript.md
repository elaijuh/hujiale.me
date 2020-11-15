+++
categories = ["tech"]
date = "2015-01-31T23:07:59+08:00"
draft = false
slug = ""
tags = ["javascript"]
title = "Scope, the most important thing in JavaScript"

+++

cope and scope chain, the most important mechanism in JavaScript, are barely clearly explained though specified in ECMA-262 version 5.1. Without scope mechanism, there wouldn't be closure and functional programming. This post is aim to elaborate what happens behind from the very beginning when control enters the global code to execution end.

<span class="more"></span>

Firstly, for easy literal, let's agree on the following abbreviation:
`GE`: Global Environment
`VE`: Variable Environment
`LE`: Lexical Environment
`OLE`: Outer Lexcial Environment Reference
`ER`: Environment Record
`EC`: Execution Context
`go`: global object (window for browser, global for NodeJS)

Take following **foo.js** as example, let's dive into it line by line
 
```javascript
// foo.js
var a = 1;
function Foo(b) {
	b = 3;
	console.log(b); // 3
	console.log(a); // 1
}
console.log(a);
Foo(2);
```
- Step 1: Control enters into global code
Every javascript file is executed from global code, when the control enters into the global code (before executing the code ), a global EC will be created and push into the EC stack. `this` is set to `go`. `VE` is set to `GE` and associated with global `EC`. `LE` is set to `GE` and associated with global `EC`. `GE` is a javascript internal object which contains `ER`(bind with `go`) and `OLE`(initialized to null). So at this stage, the global `EC` looks like:

```javascript
// step 1
GlobalEC: {
	VE: { 
		ER: { window: go }
		OLE: null
	},
	LE: {
		ER: { window: go }
		OLE: null
	}
	this: window
}
```

- Step 2: Declaration Binding Instantiation for global EC
After the execution context is created, control will scan the code (not execution) and bind each function declaration and variable declaration. At this stage, `VE` and `LE` will be augmented the same way, ie. `VE` is same as `LE`.  So the global EC looks like:

```javascript
// step 2
GlobalEC: {
	VE: { 
		ER: { window: go, a: undefined, Foo: foo }
		OLE: null
	},
	LE: {
		ER: { window: go, a: undefined, Foo: foo }
		OLE: null
	}
	this: window
}
```

The tricky part is when control scans to `Foo`'s function declaration, it will create a new function object `foo` and associate it with `Foo`. The common internal properties and special internal properties for function will be assigned to `foo`. One of these internal properties is `[[scope]]`, from here the scope chain mechanism starts. `foo.[[scope]]` is set to current `EC.LE`

- Step 3: Executing the global code 
After the control finishes scanning the code in step 2, it starts to run the code line by line. During the execution, `LE` will be updated with the value bound to the variable in `ER`, but `VE` remains unchanged. And remember `foo.[[scope]]` is set to `LE` by reference, so it will be updated accordingly. So before the control enters `Foo`'s code by calling `Foo()`, the `GlobalEC` looks like:

```javascript
// step 3
GlobalEC: {
	VE: { 
		ER: { window: go, a: undefined, Foo: foo }
		OLE: null
	},
	LE: {
		ER: { window: go, a: 1, Foo: foo }
		OLE: null
	}
	this: window
}
```

- Step 4: Control enters into Foo's code
By calling `Foo()`, the control enters Foo's code. Much like the control enters into global code, it will create a `EC` for `Foo` and push it into the EC stack on the top of global EC. Then it will create `VE` and `LE` with a null `ER` associated. The tricky difference is `OLE` is set to `foo.[[scope]]` which is global `EC.LE`. The scope chain is established right from here. At this stage, `FooEC` looks like:

```javascript
// step 4
FooEC: {
	VE: { 
		ER: null
		OLE: foo.[[scope]] // GlobalEC.LE
	},
	LE: {
		ER: null
		OLE: foo.[[scope]] // GlobalEC.LE
	}
	this: window
}
```

- Step 5: Declaration Binding Instantiation for Foo EC
Like it does in global code, the control will scan the function body before executing it, to bind the function declaration and variable declaration into `ER`. Additionally for function code, the formal parameters will be bound to `ER`.

```javascript
// step 5
FooEC: {
	VE: { 
		ER: {b: 2}
		OLE: foo.[[scope]] // GlobalEC.LE
	},
	LE: {
		ER: {b: 2}
		OLE: foo.[[scope]] // GlobalEC.LE
	}
	this: window
}
```

- Step 6: Executing the Foo code
After scanning the function code, the control starts to execute the function line by line. During this stage, `LE` is likely to be updated while `VE` remains unchanged. 

```javascript
// step 5
FooEC: {
	VE: { 
		ER: {b: 2}
		OLE: foo.[[scope]] // GlobalEC.LE
	},
	LE: {
		ER: {b: 3}
		OLE: foo.[[scope]] // GlobalEC.LE
	}
	this: window
}
```
When control comes to `console.log(b)`, it will first find the variable `b` in `LE.ER` and value 3 is retrieved. When control comes to `console.log(a)`, it will first try to find the variable `a` in `LE.ER`. Unluckily , this time there is no `a` to be found, the control keeps finding `a` in `LE.OLE` which is `GlobalEC.LE`, finally `a` is found in `GlobalEC.LE.ER` with value bound to 1.

---

[ecma-262 5.1 sec-13.2](http://www.ecma-international.org/ecma-262/5.1/#sec-13.2)

[ecma-262 5.1 sec-10.4.3](http://www.ecma-international.org/ecma-262/5.1/#sec-10.4.3) 