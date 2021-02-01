---
layout: post
title:  "All about annotations and annotation processors"
excerpt: "Understanding what annotation processors are, and how they work"
---

![Annotation processing rounds]({{site.baseurl}}/assets/images/annotation-processing-rounds.png)

Annotations provide metadata to our code. This metadata is only relevant to the scope of development and to the developer. In the absence of an annotation, this metadata information would have to be “squeezed” in our code by other means, such as naming conventions for example. To save the source code from such travesties, Java came up with the `Annotations API` in `JAVA 1.5` or `version 5` (`Java 1.6` brought complete annotations support). With the help of annotations, we can include this metadata information in different “elements” of our source code. These elements can be any of the following:
* Classes
* Interfaces
* Class or interface properties
* Class or interface methods
* Variables
* Other first class citizens of your language

After you mark you elements with the desired annotations, the class goes to the `javac` (Java Complier) as per the usual business. As expected, `javac` is unaware of these annotations and it needs some help in unfolding the meaning if these alien entities. That is where annotation processors come into play. Annotation processors unfold the true intent of their respective supported annotation and eventually passed the resulting class back to the `javac`. At this point `javac` is stuck in a loop, where each and every annotation must be resolved before it can produce the desired `java classes`. We will discuss in detail this loop and the entire cycle in detail later in the article. Let us first try to understand what annotations are and how you can create your own annotations.

# What are annotations?
To understand what are annotations and why we need them, let us observe a case. Let’s assume we have two `classes`, `A` and `B`. Both of them have a overloaded `function` called `veryLongNameFunction()`.
```kotlin
class A {
  fun veryLongFunction() {
    print("function called in class A")
  }
}
```
```kotlin
class B: A {
  fun verylongFunction() {
    print("function called in class B")
  }
}
```

On calling `b.veryLongFunction()`, class `A` function would be called. That happens because of our `typo` in class `B`’s function name. Needless to say, this will lead to a `run time error` which is expensive. To turn it into a `compile time check`, we can mark this method in class `B` with an annotation which can explicitly state that this is an `overrided` method of its `super class`.
```kotlin
class B: A {
  @override
  fun veryLongFunction() {
    print("function called in class B")
  }
}
```
Now, we will not get a `run time error` as our `compiler` will throw an `error` if we will try to use our `override` method by using `b.verylongFunction()`, as it is not overriding anything. This is the power of annotations. They can be used in various ways, in our case for instance, we used an annotation as an explicit markup. You can use annotations for code generation as well, which is what most of the external libraries do when they offer you a set of annotations to sprinkle around in your codebase. With the help of these annotations, the library’s annotation processor will kick in during compile time and it will produce all the meta data and generate new code required for the library to work with, making your job as a developer easier. One such instance of such a library is Dagger(2.0). The only way dagger is able to create a “Directed Acyclic Graph” of your dependencies is  with the help of all the annotations the library requires you to put in your codebase, such as `@inject`, `@provides`, `@module`, etc.

Now that we are aware of what annotations are and why they were needed in the first place, lets see how we can create our custom annotations and use them in our codebase.

# How to create a custom annotation
Let us suppose the name of our custom annotation is `myAnnotation`. In order to create our custom annotation, we have to provide some `meta information` about our custom annotation. This meta information will explain the retention policy and the element to which our annotation will abide. This meta information can be added through the help of, you guessed it, annotations. Here is what these meta-annotations look like:
```kotlin
@Target(ElementType.TYPE, ElementType.Class)
@Retention(RententionPolicy.RUNTIME)
```

In `Target`, we specify the type of element we will attach our annotation, such as `variable`, `class`, etc.
In `Retention`, we specify how long our annotation will be retained, i.e. either until `SOURCE`, `CLASS`, or `RUNTIME`. Retaining a annotation until runtime can allow us to use `reflection` to read the annotation’s data.

There are three types of annotations we can create based on our needs:

1. Marker annotation, as it does not have any value:
```kotlin
@Target(ElementType.TYPE)
@Retention(RententionPolicy.RUNTIME)
@interface myAnnotation {
}
```
2. Single value annotation, as it has a function that returns one value:
```kotlin
@Target(ElementType.TYPE)
@Retention(RententionPolicy.RUNTIME)
@interface myAnnotation {
  fun myField(): String
}
```
If you dont want to use the argument name, then name the function as value:
```kotlin
@Target(ElementType.TYPE)
@Retention(RententionPolicy.RUNTIME)
@interface myAnnotation {
  fun value(): Int
}
```
3. Multi value annotation, as it has multiple functions that return a value:
```kotlin
@Target(ElementType.TYPE)
@Retention(RententionPolicy.RUNTIME)
@interface myAnnotation {
  fun myFirstField(): String
  fun mySecondField(): Int
}
```

So now in order to annotate our element with our custom annotation, all we need to do is this:
1. For Marker annotation:
```kotlin
@myAnnotation
class A {}
```
2. For Single value annotation(named argument):
```kotlin
@myAnnotation(myField = "String")
class A {}
```
For Single value annotation(unnamed argument, where the name of the function is “value”):
```kotlin
@myAnnotation(10)
class A {}
```
3. For Multi value annotation:
```kotlin
@myAnnotation(myFirstField = "String", mySecondField = 1)
class A {}
```

Notice how our `single value` and `multi value` annotation form can take arguments. These `parameters` have the same name as the function name defined in the `interface` and the data type they accept is the one the function returns. These arguments can be used by the annotation processor as `flags` or `configuration parameters` for any purpose you deem fit. You can also provide default values to these arguments in case if the arguments are not provided.

There are certain rules you should be aware of if you are going to define your own custom annotations:
* Annotations cannot participate in inheritance.
* Annotation’s functions should not contain any arguments.
* Annotations cannot be generic or specify a throw clause.
* Annotations must return: an enum, primitive type or an annotation, String, or Class object. They can also return an array of these types.

Now let’s talk about annotation processors, what they are, what they do, how they do it, and how can we create our own custom annotation processor.

# Annotation Processors
Annotation processors work or kick in inside `javac`(Java Compiler). We don’t have to opt into this. Just by having an annotation processor on our `classpath` when our code is compiling, our annotation processor will run. It does so with the help of something called `Service Loader`, which finds the processors, and processors is the class that we will implement in order to create our custom one.

1. Let us create our own processor called “CustomProcessor” as below:
```kotlin
class CustomProcessor: AbstractProcessor() {
}
```
2. The first function that we will implement is called `getSupportedSourceVersion()`, which will tell the level of `java` coming in and the level of `java` going out. It is a good idea to always support the latest version that our `javac` is running on.
```kotlin
class CustomProcessor: AbstractProcessor() {
  @Override 
  fun getSupportedSourceVersion(): SourceVersion {
        return SourceVersion.latestSupported()
  }
}
```
3. The second function that we will implement is called `getSupportedAnnotationTypes()`, which tells `javac` what the fully qualified names of the annotations we care about are. So `javac` will only call our processor for the annotations that we tell it to.
```kotlin
class CustomProcessor: AbstractProcessor() {
  @Override 
  fun getSupportedSourceVersion(): SourceVersion {
        return SourceVersion.latestSupported()
  }
  @Overried 
  fun getSupportedAnnotationTypes(): Set<String> {
        return Collections.singelton(Example.class.getName())
  }
}
```
4. The third function is the most important one is called `process()`:
```kotlin
class CustomProcessor: AbstractProcessor() {
  @Override
  fun getSupportedSourceVersion(): SourceVersion {
        return SourceVersion.latestSupported()
  }
  @Overried
  fun getSupportedAnnotationTypes(): Set<String> {
        return Collections.singelton(Example.class.getName())
  }
  @Override 
  fun process(annotations: Set<?: TypeElement>, env: RoundEnvironment): Boolean {}
}
```
Lets us understand what happens inside the process function.
    1. We query our environment to get all the elements that our annotated with our annotation:
    ```kotlin
    @Override 
    fun process(annotations: Set<?: TypeElement>, env: RoundEnvironment): Boolean {
        // We query the types that are annotated with our annotations.
        // The returned elements are the ones marked with our Example Annotation.

        let elements: Set<?: Element> = env.getElementsAnnotatedWith(Example.class)
    }
    ```

    2. We can use these collected elements, which could be `classes`, `variables`, `functions`, use libraries called `javaPoet` and produce `java classes`.
    ```kotlin
    @Override 
    fun process(annotations: Set<?: TypeElement>, env: RoundEnvironment): Boolean {
        let elements: Set<?: Element> = env.getElementsAnnotatedWith(Example.class)

        for(element in elements)
            print(element)

        return false
    }
    ``` 

    > One cool thing about annotation processors, besident code generation, is that they run in their own `JVM`. Though you can not access source files and its dependencies as they are being compiled as well during the time our annotation processor is working, however, we can bring our own set of dependencies. This means that you can create `models`, `factories`, etc. anything to make your annotation processor module clean and solid. This code will not contribute to the generated code and will only be scope to the `JVM` in which your annotation processor will be running on.

5. The fourth function that is present in the AbstractProcessor is called `init(processingEnv: ProcessingEnvironment)`. As expected, in `init` we do all the initialization for our custom processor. `ProcessingEnvironment` is yet another interface which we will not discuss here, however it has two important functions that we should talk about:
    First is `fun getMessenger(): Messenger`, which returns us a `Messenger` object. With the help of `Messenger`, we can communicate the `warnings` or `errors` back to the third party developer who is compiling in case there is an issue in the `process()` function. This is a good practice as in its absence, if an `exception` occurs, our `jvm` in which our custom annotation processor is running will simply crash and produce a `backstack dump`, which is not useful to the developer. We can also control how the third party developer will use our annotations by throwing `errors` that will force them to follow the rules we will lay down with respect to how to use our annotations.

    Second is `fun getFiler(): Filer`, which returns us a `Filer` object. In short, `Filer` helps us to actually create and write `java files`.

The last two we should look at are fun `getElementUtils(): Elements` & `fun getTypeUtils(): Types`. These methods return utility classes that we can use to work with elements of different types that we will receive based on where our annotation is placed on. An analogy here would be that of `XML`, which has the concept of parent and children elements. Similarly, in java we define our class as `TypeElement`, variables as `VariableElement`, and functions as `ExecutableElement`.

So the complete code for our custom annotation processor will look like this:
```kotlin
class CustomProcessor: AbstractProcessor() {
  @Override 
  fun getSupportedSourceVersion(): SourceVersion {
    return SourceVersion.latestSupported()
  }

  @Overried 
  fun getSupportedAnnotationTypes(): Set<String> {
    return Collections.singelton(Example.class.getName())
  }

  @Override 
  fun process(
    annotations: Set<?: TypeElement>,
    env: RoundEnvironment
  ): Boolean {
    let elements: Set<?: Element> = env.getElementsAnnotatedWith(Example.class)

    for(element in elements)
        print(element)
  }

  @Override 
  fun init(processingEnv: ProcessingEnvironment) {}
}
```

Now let’s talk about how different annotation processors works together in `javac` with harmony.

# Processing Rounds
There is a cycle that is followed by `javac`, which allows every annotation processor to generate code for the annotations it is open for. This is the order of that flow:

1. We mark the element that our annotation is open to.
2. `Javac` will start compiling. `Javac` is already aware of all the other annotation processors presents, as we have put them in the `classpath` which will be picked up by the `ServiceLoader`.
3. `Javac` will produce the `.class` files in the first processing round. As the `javac` has already processed the `.java` files, it is already aware of the annotations which were used. It will pass the control to the respective annotation processor now.
4. Now the annotation processor will start to do its job. Here, it may or may not produce additional `.java` files. For the sake of argument, let’s say our customer annotation processor generated a `.java` class which contains annotations to be used by some other annotation processor.
5. At this moment, all the original `.java` files have been compiled. However, because of a `.java` file generated by our customer annotation processor, `javac` will detect this and will start another processing round.
6. This cycle will keep on going until no more `.java` files have been generated and detected by the `javac`.
It is a good idea to create domain specific models for the Elements we will process in the custom annotation processor’s process() function as it will make working on these elements easier. For example, if our annotation processor has a constraint that says “all the annotated classes must be in the same package”, then we can simply refer to the model’s package property instead of working on the element itself.

A fun closing remark: Remember in the beginning we said that `ServiceLoader` is the one that will find our custom annotation processor and load it into the `javac`. Well in order for the `ServiceLoader` to do that, we have to create a certain file in a repository in our project. However, we can use a cool annotation processor called `AutoService`. All we need to do is mark our custom annotation processor’s class with `@AutoService(Processor.class)` and it will generate that repository in our project.