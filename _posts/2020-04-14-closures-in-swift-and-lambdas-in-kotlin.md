---
layout: post
title:  "Closures in swift and lambdas in kotlin"
---
Kotlin and Swift are currently the top two choices of language for Android and iOS respectively. Both have been successfully adopted by their respective communities of developers who use these sharp languages to create some amazing pieces of work. Even though these two languages have their own trademark, underneath it is all the same with different names for legal purposes:
* Structures offered by Swift can be created as Data Structures in Kotlin by enforcing the kotlin classes to follow some rules.
* Protocols in Swift is Interfaces in Kotlin.
* Both languages allow functional programming.
* Kotlin has sealed classes, however Swift has great Enum support.
* Closures in Swift are Lambdas in Kotlin.

You get the idea. Though there are some pieces of differences here and there, these differences do not matter in the grand scheme. Today, we will take a look `closures` in swift, which are blood-brothers to “lambdas” in kotlin.

# Closures in Swift
A closure in Swift is a anonymous function which can be passed around in the scope. What this means is that functions are treated as first class citizens, just like variables or objects in Object Oriented Programming(OOP) environment. A function in swift that accepts a function as one of its parameter looks like this:
```swift
func exampleFunction(
    param1: String, 
    param2: String, 
    ourFunc: (Int, Int) -> Int) {
        ...
    }
```

where `ourFunc` is our `function` passed as a `parameter` to `exampleFunction`. `ourFunc` will accept two `params` of its own of the data type `Int` and would return an `Int` type. Now to make it into a `closure`, we have to call our `exampleFunction`. In order to do that, we will go through certain steps and transformations:
1. we will call our function and provide all the params:
```swift
exampleFunction("param1", "param2", { (no1: Int, no2: Int) -> Int in
    // Here we are inside our 'ourFunc' can use no1 & no2 parameter
    return no1 + no2
    }
)
```

2. Here’s where closures start to become a bit tricky. The swift compiler is smart enough to infer the type of the parameters that we will pass in the ‘ourFunc’, so omitting the data types will change nothing:
```swift
exampleFunction("param1", "param2", { (no1, no2) -> Int in
    return no1 + no2
})
```

3. again, we dont need to explicitly define the return data type as swift compiler can infer that as well:
```swift
exampleFunction("param1", "param2", { (no1, no2) in
    return no1 + no2
})
```

4. We also dont need to explicitly say `return`, as that can also be infered like below:
```swift
exampleFunction("param1", "param2", { (no1, no2) in no1 + no2 })
```

5. Closures in swift allow us to pass anonymous parameters, as `$0` for first param, `$1` for second param, and so on. So we can further simply our function call to:
```swift
exampleFunction("param1", "param2", { $0 + $1 })
```

6. Now lastly, swift allows us with a feature where if the closure is defined as the last parameter in a function, i.e. a trailing closure, we can take it out of the function’s parameter parentheses like below:
```swift
exampleFunction("param1", "param2") { $0 + $1 }
```

As you can imagine, someone who is not aware of closures or lambdas might get confused if they sees the form define in the 6th point. So any of the forms mentioned above can be used as they all mean the same thing. Its simply an anonymous function being passed as a parameter.

Now let us take a look at Lambdas in kotlin and then you will realize how both closure and lambdas are the same thing.

# Lambdas in Kotlin
A lambda expression is nothing but a shorthand way to define the structure of a function. Lets say we have a function as below:
```kotlin
fun myFunction(param1: String, param2: String): String
```

The same function can be defined as a Lambda expression as below:
```kotlin
(String, String): String
```

A lambda expression tells us:
* What the types of parameters are and how many parameters do we have.
* What is the return data type of the function.

Just like in Swift, functions are treated as a “First class” citizens in kotlin. Which means that functions can be passed as parameters to another functions, functions can be the return type of another function and functions can be saved in a variable. Now lets see how we will pass a lambda function to a normal kotlin function as one of its parameter.

1. Lets say that our function will contain one `String` parameter and one `Lambda` function parameter as below:
```kotlin
fun myFunction(param1: String, lambda: (a: Int, b: Int) -> String ): String {
    // We assume that our lambda function will return "5".
    val stringReturnedFromLambda = lambda(2, 3)

    return param1 + stringReturnedFromLambda
}
```
> Note: It is not necessary to name our parameter “lambda”. It is being done to simplify.

2. Now when we will call myFunction, we will have to provide this lambda function on its calling. This lambda function simply has to follow the contract specified in the function definition. So we will get something like below:
```kotlin
muFunction("myString", { a, b ->
    val sum = a + b

    return sum.toString()
})
```

3. If we only have one parameter in our lambda, for example:
```kotlin
fun myFunction(param1: String, lambda: (a: Int) -> String ): String {
    val stringReturnedFromLambda = lambda(2)

    return param1 + stringReturnedFromLambda
}
```
Then we dont need to explicitly define the parameters `a` and `b` like we did in #2. Below representation would be fine as well:
```kotlin
muFunction("myString", { 
    return it.toString()
})
```
As you can see, for one parameter, we get something called `it`, which is nothing but our handle to the `lambda parameter`. You are free to give a name to this it, just like we did before:
```kotlin
muFunction("myString", { a ->
    return a.toString()
})
```

4. Now for the trailing lamda part, which means if the last parameter of a function is a lambda expression, then we can bring it out of the parameter parenthesis just like we did in case of closures:
```kotlin
muFunction("myString") { a ->
    return a.toString()
}
```
We can shorten it by using it :
```kotlin
muFunction("myString") { it.toString() }
```

As you see, both Closures and Lambdas are made to solve the same problem: Passing anonymous functions as parameter to another function. They might have different names because of belonging to two different languages, however they are one and the same thing.