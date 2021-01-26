---
layout: post
title:  "Navigation in a pure jetpack compose project - part #1"
---
![Compass]({{site.baseurl}}/assets/images/compass.jpeg)

Jetpack compose is the new kid in the block. Though its relatively new, compared to the traditional android UI toolkit, soon will come those days when projects will start migrating to jetpack compose, and more & more companies will put it as a requirement for their potential android developer. As of the publishing date of this article, the argument "will composables completely replace fragments or views?" has not been achieved yet. There are certain things, on the other hand, which have been achieved with jetpack compose, one of which is navigation among different composables. Let's take a look and see how jetpack's navigation component library works seamlessly with jetpack compose.

## Adding the dependencies
First thing that you will need is to bring in the following jetpack navigation component dependency designed specifically  by keeping jetpack compose in mind:

{%gist bf2cb2d985e973dffa3b072438182931%}

Now that we have our dependencies in order, let's start with discussing how navigation actually happens among composables.

## NavController - The good Ol' friend
If you have used navigation component in the past, then you are going to feel right at home (well almost). Similar to legacy UI toolkit, we are going to use `NavController` here as well, as it will act as our central API. Navigation component will also manage our `backstack` and add or remove the composables from it.
Simply add the `navController` to your composable as shown below:

{%gist 1427c1fb595d8f41159d2bd5ad9cbc9d%}

As recommended in the navigation for compose docs, it's a good idea to place this `navController` at a hierarchical level where other composables can easily access it. This way you can update the composables that are outside your screen (i.e. manage the ones present in the `backStack`)with the help of `navController` and the state it provides via `currentBackStackEntryAsState()`.

However, if you go on a refresher course on navigation component, you would soon realize that every `navController` need its partner-in-crime, the `NavHost`. The `NavHost` is responsible for linking the `NavController` with a navigation graph that specifies all the composable destinations that you should be able to navigate between. As you navigate to different composables, the content of the `navHost` gets switched with the destination you reached at, or to say that your `navHost` gets `recomposed`. In the case of legacy UI toolkit, this NavHost was a `FragmentContainerView`. Each composable in your navigation graph acts as a destination and is reachable via a `route`.

> A `Route` is a `String` that defines a path to your composable. Think of it as an implicit deep-link that leads to a specific destination in your application. Each destination in your navigation graph, or each composable, must have a unique `route` to avoid collision.

To create the `NavHost`, you will need the `navController` that we previously 'remembered' in our composable, and the name of the `startDestination` composable. With the help of a little Kotlin DSL magic, here is how to create a `NavHost`:

{%gist b7330b7900b2a826a55be60ffe19740e%}

Now that all the elements of navigation component are setup, comes the time to actually navigate to another composable. In order to carry out this transaction, simply use `navigate()` method on the `navController` instance as follow:

{%gist 06126ac2b49746bbcb5be61ec7980993%}

> Keep in mind composables `recompose` multiple times. So make sure to use the `navigate()` method in a `callback` and not in the composable itself.

NavController also provides you with the APIs to manage the `backstack` with composables as well, similar to how it does for the the native UI toolkit. For instance, in order to pop the `backstack` everything up to the mentioned destination, you can do the following:

{%gist 67ccdc96aefba81b64e0b7a857a21f54%}

Similarly, you can also include the `inclusive` flag, which will also pop the mentioned destination from `backstack` as well, in our case the `home` composable:

{%gist ae11805ad9b6f3cb7e172ec536552c08%}

Much like how activities or fragments provide us with managing their various instances in the `backstack` with the help of `launch modes`, similarly composables can make use of those `launch modes` as shown below:

{%gist 9019ea0e355544450a6a6377d2f2ff03%}

## Passing arguments between destinations
In order to pass arguments between our composables, we would need to modify the `routes` we defined in our `NavHost`. This is eerily similar to how we define `PATHS` in a retrofit service, as shown below:

{%gist 92f01633dec02fa3c3fedcc9bba97ec9%}

Now in order to pass arguments, simply switch the `userId` parameter in the `navigate()` method call and pass in the actual value as show below:

{%gist 104c1804d20ab496e76e496b934854c3%}

## Adding Deep Links to composables
Jetpack compose UI would not have a good interoperability with the existing UI toolkit unless it solves the problem of providing `deeplinks` to the composables. Follow the steps below in order to add an implicit deep link to your composable destination:

{%gist 73ba4eeee31e80dba33c6bc00a6a4d9c%}

One last step to complete the `deeplinks` setup would be to add the `<intent-filter>` to our `manifest.xml` file. Add the following to your activity to allow 3rd party apps to open your application:

{%gist 184f5825fff3e7ca5bc4d5ea0e488990%}

In part #2, we will see how to implement `BottomNavigation` in jetpack compose and how to use navigation component with it to navigate to different composables.