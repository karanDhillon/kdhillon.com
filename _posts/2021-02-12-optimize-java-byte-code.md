---
layout: post
title: "Optimize java byte code"
excerpt: "Best practices that should be followed"
---

The intention of this article will be to convince android developers (Java or Kotlin) to incorporate the following mentioned practices in how they write their source code in their personal or work projects in order to produce optimized java byte code. The benefits that the developers will reap from these practices are the following:
* Reduce total method count, and make optimized use of `65k` method count limit imposed by `dex`.
* Save **space** and **time** by removing redundant bridging logic from the byte code.
* Save **space** and **time** when using `Collections` library.
* Understand how new features introduced by latest Java versions are backward compatible.

Lets start the discuss by first observing how byte-code world is different than the source code world, and how are perceptions at the higher abstracted source code level actually lies to us. Below we have an empty Java class called `Example.java`, whose source code looks something like this:

**Example.java**
```
class Example { }
```
After we compile the above class using `javac`, and generating `Example.class`, we can sort of peek into this new generated `Example.class` with the help of a tool called `javap`. `javap` ships within the `JDK` itself. Running `javap Example` will produce the following result:

**Example.class**
```
class Example {
  Example();
}
```
Right away we start seeing the differences between source code and the transformed byte code. Even though we did not define any `constructor` for our `Example` class, there is a constructor method `Example()` present in its byte code.

`dex`, which takes in the Java byte code and converts it into dex code is present in every android SDK, and one add it to their environment's classpath for quick access. Running `dex` on `Example.class` will create `Example.dex` as expected. Peeking directly into `Example.dex` is not quite possible, however another tool called `dexdump` (also present in android SDK, in platform specific version) to fetch the information that we need for our `Example.dex` file. Upon running `dexdump Example.dex`, we get the following:

**Example.dex**
```
Processing 'Example.dex'...
.
.
.
method_ids_size     : 2
.
.
.
```
Redundant data has been redacted from the dex dump, because what we value currently is the total method count of our `Example.dex`, which is indicated by this field called `method_ids_size`. Quite to our surprise, there are `2` methods present in our final `Example.dex` file. We can view these methods in the dex dump using `dexdump`, and they are:
```
Example <init>()
java.lang.Object <init>()
``` 
So in reality, our compiled `Example.class` contained not `1`, rather `2` methods, and this would've been unknown to us if we never actually dumped this class using `dexdump`. To understand the source of these methods, we run `javap -c Example`:

**Example.class**
```
Compiled from "Example.java"
class Example {
  Example();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
}
```
What we are seeing is java byte code syntax, which follows `opCode argument1 argument2` format. As `JVM` makes use of `local variable array` and `operand stack`, `0: aload_0` simply indicates to `load` value `0` in the `array`. What we are after is the statement `1: invokespecial #1` which calls the `Object` class `constructor`. The reason why this exists is the fact that every class call the `super` of its parent class, which in this case is `Object` class. To sum up, our 'empty' `Example.java` eventually lead to contributing `2` items in the total method count of the `65k` dex limit. This was a small example to show how our perception of the source code starts to become false at the byte code level.

> `invokespecial` is `1` out of `4` opcodes in its category, others being `ivokeinterface`, `invokevirtual`, `invokestatic`.

A nice segway from here would be about what should be done at source code level, so that the byte code is optimized, and so is the dex code. One such instance is the case of nested classes, which were introduced after Java `1.0`. We often encounter code in our projects that look like below:

**ItemsView.java**
```
class ItemsView {
    private class ItemsAdapter {
    }
}
```
This is a pretty common instance in android world, where the `adapter` implementation is nicely present in its `view` class scope. Lets proceed further by compiling our `ItemsView.java` using `javac` and running `javac ItemsView.java`. What we get is not `1` class file, but `2` class files, namely:
1. `ItemsView.class`
2. `ItemsView$itemsAdapter.class`

To reference back to how we perceive source code, our above example clearly states that the concept of nested classes only exist at the source code level, and java does not really have nested classes (at the byte code level), otherwise we would have only seen `ItemsView.class` file. using `javap` on both of these classes, we can see their constructs:

**ItemsView.class**
```
class ItemsView {
  ItemsView();
}
```

**ItemsView$ItemsAdapter.class**
```
Compiled from "ItemsView.java"
class ItemsView$ItemsAdapter {
  final ItemsView this$0;
}
```
This means that even though we nesting two classes as shown above, what we are practically getting are two classes next to each other in their package space as shown below:
```
public class ItemsView{
}

class ItemsView$ItemsAdapter {
}
```
However, we know that an inner class, such as `ItemsView$ItemsAdapter` can access the private members defined in the scope of its outer class, which is `ItemsView` in this case. So how is it possible? Clearly on the byte code level, there is no such thing as nested classes. So how does byte code achieves the same behavior?
```
public class ItemsView {
    private static String displayText(String item) {
        return "";
    }

    private class ItemsAdapter {
        void bindItem(TextView tv, String item) {
            tv.setText(ItemsView.displayText(item));
        }
    }
}
```
The answer can be easily found by running `javap -c ItemsView$ItemsAdapter`:

**ItemsView$ItemsAdapter.class**
```
class ItemsView$ItemsAdapter {
    void bindItem(android.widget.TextView, java.lang.String);
    Code:
        0: aload_1
        1: aload_2
        2: invokestatic #3 // Method ItemsView.access$000:...
        5: invokevirtual #4 // Method TextView.setText:...
        8: return
}
```
The opcode that we are interested in is `2: invokestatic #3 // Method ItemsView.access$000:...`. We never created any `access$000` method for `ItemsView` class, but clearly its there. This is what is known as synthetic accessor method. Running `javap -p ItemsView` further proves the presence of accessor method `access$000` in its byte code:

**ItemsView.class**
```
class ItemsView {
    ItemsView();
    private static java.lang.String displayText(...);
    static java.lang.String access$000(...);
}
```
Its not hard to imagine what's going on here. Due to the fact that there is no class nesting at byte code level, and to solve the problem of accessing a private member of an outer class, an accessor method is generated automatically which simply serves the purpose of delegation. At source code level, this will simply mean the following:

**ItemsView.java**
```
public class ItemsView {
    private static String displayText(String item) {
        return "";
    }

    static String access$000(String item) {
        return displayText(item);
    }
}

public class ItemsView$ItemsAdapter {
    void bindItem(TextView tv, String item) {
        tv.setText(ItemsView.access$000(item));
    }
}
```
The question that should be asked at this point is: why should I bother about these synthetic accessor methods and the way JVM does its job? The answer: run `dx` and see for yourself. The accessor method will still be present in the final `dx` class file, and is contributing to your `65k` method count limit. `Proguard` or `D8` can sometimes eliminate these synthetic accessor methods, but there is no guarantee. 

Now comes the interesting part: How do I stop the generation of synthetic accessor methods and save my `65k` method count limit? The simplest solution to this problem is to mark the `private` outer class member to `package-scoped`:

**ItemsView.java**
```
public class ItemsView {
    static String displayText(String item) { // No more private
        return "";
    }

    private class ItemsAdapter {
        void bindItem(TextView tv, String item) {
            tv.setText(ItemsView.displayText(item));
        }
    }
}
```

Now upon running `javap -p ItemsView`, you will notice there are no accessor methods present anymore.

Moving along the same path, `generics` is another case marked by these synthetic accessor methods. Below is a pretty common use case, where we have a single access method interface, commonly known as `SAM`:

**Callbacks.java**
```
interface Callback<T> {
    void call(T value);
}

class StringCallback implements Callback<String> {
    @Override public void call(String value) {
        System.out.println(value);
    }
}
```

Running `javap -p StringCallback` shows us how every method that takes in a generic of type `T` produces two methods:
• `public void call(java.lang.String)`
• `public void call(java.lang.Object)`

**StringCallback.class**
```
Compiled from "Callbacks.java"
class StringCallback implements Callback<java.lang.String> {
  StringCallback();
  public void call(java.lang.String);
  public void call(java.lang.Object);
}
```
This is in essence what `type erasure` looks like. When the method `call` is actually called, the `call(java.lang.Object)` is initially called which first type-casts the argument passed to it, and then delegates the call to the `call(java.lang.String)` method as show below:

**StringCallback.class**
```
Compiled from "Callbacks.java"
class StringCallback implements Callback<java.lang.String> {
  StringCallback();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void call(java.lang.String);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: aload_1
       4: invokevirtual #3                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       7: return

  public void call(java.lang.Object);
    Code:
       0: aload_0
       1: aload_1
       2: checkcast     #4                  // class java/lang/String
       5: invokevirtual #5                  // Method call:(Ljava/lang/String;)V
       8: return
}
```
This is also true for the case where a methods returns a generic type of value `T`. Covariance also plays a role in generating another method for each of the sub-types of this return type `T`, as shown below:

**Providers.java**
```
interface Provider<T> {
    T get();
}

class TextViewProvider implements Provider<View> {
    @Override View get() {
        ...
    }

    @Override TextView get() { // TextView extends View, and can be passed as the return type
        ...
    }
}
```
Running `javap -p TextViewProvider` will show `3` methods, one of `Object` type, one for `TextView`, and another for `View`. `Proguard` or `D8` can again sometimes help in removing the `Object` method as its simply doing the type casting, however this is not guaranteed, and is something that should be kept in mind.

There are some other micro-optimization tips that can be followed in tandem with optimizing our byte code to generate code that executes faster:
* Use `SparseArray` instead of `HashMap`. Object overhead is `3` times more in the case of `HashMap`, and there is more re-directing in a `HashMap`. Only applies when data to be stores in small and in the range of hundreds, and not thousands.
* Instead of initializing an empty list, try to initialize the list with a fixed size:
```
public List<String> names(List<User> users) {
    List<String> names = new ArrayList<>(); // bad practice
    for(User user: users) {
        ...
    }
}
```
In `JVM`, the default size for an empty `ArrayList` will be size `10`. So if our `users` is greater than `10`, then at the `11th` entry, a new `List<User>` will be created and the contents of our original list will be copy-pasted in the new one. A good solution to the above will be the following:
```
public List<String> names(List<User> users) {
    List<String> names = new ArrayList<>(users.size());
    for(User user: users) {
        ...
    }
}
```
* Calling an instance method multiple times. Take a look at the example below:
```
@Override protected void onCreate(Bundle bundle) {
    super.onCreate(void);

    setTitle(getResources().getString(R.id.title));
    getWindow().setStatusBarColor(getResources().getColor(R.color.blue));

    headerOffset = getResources().getDimensionPixelSize(R.dimen.header_offset);
    headerTop = getResources().getDimensionPixelSize(R.dimen.header_top);
    animationHeight = getResources().getDimensionPixelSize(R.dimen.animation_height);
}
```
`getResources()` is getting called at multiple places, and every call will have its overhead. A better solution to the above would be:
```
@Override protected void onCreate(Bundle bundle) {
    super.onCreate(void);
    Resources res = getResources();

    setTitle(res.getString(R.id.title));
    getWindow().setStatusBarColor(res.getColor(R.color.blue));

    headerOffset = res.getDimensionPixelSize(R.dimen.header_offset);
    headerTop = res.getDimensionPixelSize(R.dimen.header_top);
    animationHeight = res.getDimensionPixelSize(R.dimen.animation_height);
}
```

This article was heavily inspired by Jake Wharton's talks he gave on the same topics. All the links are mentioned below:
* [Jake Wharton: Exploring Java's Hidden Costs](https://www.youtube.com/watch?v=WALV33rWye4)
* [Streamlining Android Apps: Eliminating Code Overhead by Jake Wharton](https://www.youtube.com/watch?v=b6zKBZcg5fk)