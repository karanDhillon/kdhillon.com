---
layout: post
title: "Navigation component in multi-module android apps"
---

![Nav graph]({{site.baseurl}}/assets/images/navigation-graph.png)

With the introduction of Navigation component in android Jetpack library, a ton of features were offered to the developers to ease the pain of arduous Fragment transactions and manually managing the back stack. Sure one can still do it, but given the detail of the Navigation component’s API, it would be a smart idea to let the library do the heavy lifting. 

Some of the features that navigation component offers to the developers are:
* Automatic fragment Transactions.
* Automatic Back stack management.
* SafeArgs plugin offers Null checks and Type casts for passed arguments.
* Single resource file for navigation.

This is no new information. What I want to talk about is something Google docs do not cover: How to implement Navigation Component in Multi-module applications. 

Before we discuss various cases of how your application will be configured, it would be a good idea to go through the basics of Navigation Component, which is out of scope for this article.

For our application, the architecture we follow will be pretty simple:
<center>
    <img src="{{site.baseurl}}/assets/images/ss.png" alt="App module" width="500" align="center">
</center><br/>

1. Keeping the Nav_graph in :APP module
    In our :app module, we can refer to every feature sub-module, So we can create the application’s navigation graph in their. However, this has one problem. The reason we modularize our apps is to segregate our application into small features. Our end goal is to simply  use :APP module as a place to integrate all these :FEATURE modules. If we create the whole app’s navigation in the :APP module, we will be derailing from that goal. Also, we will not be able to refer to SafeArgs generated classes because those classes will be generated per module. So if Feature1 wants to navigate to Feature2, we will not be able to make use of Feature2’s SafeArgs classes. Our whole navigation strategy would have to be explicitly defined in the :APP module itself.

2. Keeping the Nav_graph of each Feature in the respective module
    Having the entire navigation plan per feature sounds like a good idea. Every feature will be well isolated from each other and will contain all the logic of its own navigation. There still, however, exists the problem of inter-module navigation. To work around that, we can for example, use Deep links, include the sub graphs in the main application’s graph and that way we will get automatic intent-filters. (NOTE: do not forget to register the App graph in the application’s manifesto).

    However, once again, we lose the SafeArgs benefit. Plus, creating Deep links for every destination sounds cumbersome and error prone. Let us see how #3 is a better solution overall.

3. Keeping the Nav_graph in :BASE module
    Keeping the Nav_graph in the :BASE module is the best approach among all of them. Having all the feature nav graphs included in the :BASE graph opens the applications entire navigation strategy to all the feature modules. This way we get all the benefits of SafeArgs and any feature can navigate to any other features. Sounds awesome, but what is the caveat?
    FragmentNavigator makes use of Reflection when it looks for SafeArgs generated classes and argument classes. :BASE module, being unaware of all the :FEATURE modules, will render false Lint checks for all the references we will make in the :BASE module. It will compile just fine though, but we lose the Android Studio working for viewing our app’s navigation flow. A small price to pay for a big victory.