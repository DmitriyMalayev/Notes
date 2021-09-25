Definitions

`Execution Context`
When JavaScript functions run, they have an associated JavaScript Object that goes along with them which they can access by the keyword "this"

`this`
Inside of a function, "this" is the Object that represents the function's execution context 

`call`
This is a method on a function that calls the function, just like()
We provide a new execution context as the first argument, traditionally called "thisArg", and the argument you want to send to the function after the thisArg. An invocation of call looks like Calculator.sum.call(multilingualMessages, 1, 2)

`apply`
This is a method on a function that calls the function, just like()
We provide a new execution context as the first argument, traditionally called "thisArg", and the argument you want to send to the function after the thisArg. An invocation of call looks like Calculator.sum.apply(multilingualMessages, [1, 2])

`bind`
This method returns a copy of the function but with the execution context "set" to the argument that's passed to bind. 
sayHello.bind(greenFrog)("Hello") //=> "Mr. GreenFrog says *Hello* to you all."

