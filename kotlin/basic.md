### Function
normal function definition:
```
fun function_name(arg_name: argument_type, argument_name2: argument_type2): return_type {
	// function content
}

fun sum(a: Int, b: Int): Int {
	return a + b
}
```
Function with an expression body and inferred return type:
```
fun sum(a: Int, b: Int) = a + b
```
Function returning no meaningful value may return Unit or no return:
```
fun printSum(a: Int, b: Int): Unit {
	println("sum of $a and $b is ${a + b}")
}
// ommit Unit return type
fun printSum(a: Int, b: Int) {
	println("sum of $a and $b is ${a + b}")
}
```
### Variable
use `val` to define a read-only variable:
```
val a: Int = 1	// full annoucement and assignment
val b = 2	// ommit type Int, which is inferred
var c: Int	// assign later, which requires type
c = 3		// assign the corresponding type value. can only be assigned once
```
use `var` to define a mutable variable:
```
var x = 5 // Int type is inferred
x += 1
```
### String
string templates:
```
var a = 1
val s1 = "a is $a"	// use $ to refer to corresponding variable

a = 2
val s2 = "${s1.replace("is", "was")}, but now is $a." // can use expressions or operations inside ${}
```
### Nullable values
A reference must be explicitly marked as nullable when null value is possible:
```
fun parseInt(str: String): Int? { // the return type may be null, use ? to represent
}
```
### Type checks and automatic casts
The `is` operator checks whether an expression is an instance of a type. And kotlin will cast the expression to a certain type in the effective branch if possible:
```
fun getStringLength(obj: Any): Int? {
	if (obj is String) {
		// `obj` is automatically cast to `String` type in this branch
	}
	// outside the type-checked branch `obj` is still type of `Any`
}
// or
fun getStringLength(obj: Any): Int? {
	if (obj !is String) return null
	// in this branch `obj` is cast to `String` type
}
// or
fun getStringLength(obj: Any): Int? {
	if (obj is String && obj.length > 0) { // `obj` is cast to `String` on the right-hand side of `&&`
		return obj.length
	}
	return null
}
```
