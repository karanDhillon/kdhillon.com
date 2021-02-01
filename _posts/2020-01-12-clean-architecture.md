---
layout: post
title:  "Clean architecture"
excerpt: "Learn what it means to implement 'clean architecture' in the project"
---
Writing code is fun. Maintaining it is not so. Statistically speaking, most of a developer’s time goes into reading the code than writing it. It sounds counterintuitive but it’s actually not if you think about it.You can make new features after every sprint if you follow agile practices, however during that sprint period, you will still try to spend time understanding how this new feature will fit into the ‘grand scheme’ and spend most of your mind cycles thinking what needs to be changed and what doesn’t. I believe, that is where clean architecture comes into play.

To simply put, Clean architecture is just a bunch of rules and principles that needs to be followed which eventually will keep your software ‘soft’, i.e. easy to change. Now for every platform, the architecture would change its shape, however, on higher level, if two projects are to follow clean architecture, it is easy to understand both no matter on which one you are working on. This leads us to creating a higher level policy for our applications.
Out of tons of principles, the most notable and famous ones are called the SOLID principles. The acronym stands for:
```
S – Single responsibility Principle.
O – Open/Closed Principle.
L – Liskov Substitution Principle.
I – Interface Segregation Principle.
D – Dependency Inversion Principle.
```
In my personal belief, what gives clean architecture its cleanliness is `“Dependency Inversion Principle”`. In retrospect, all my problems in various projects used to happen because of low – level details creeping up into my high level policy. Change made in a small and far reached dependency somehow will start breaking how the view deals with a particular state. That happens because of these dependencies creeping their way up to places where they should not even exist. Out of these, the ones most hard to catch are called the `“Transitive Dependencies”`, but let’s come back to the topic as explaining that will be going on a tangent.

This post will not go into details about what SOLID principles are. A quick google search will suffice. I rather wants to talk about the meaty parts of the clean architecture itself. So let’s start with my favourite, which is DOMAIN.

# DOMAIN
 
Domain creates what is called the `“Enterprise Logic”` in the realm of clean architecture. This means any logic that will live in your domain will be similar throughout your suite of applications. This could be your iOS, android, flutter, react native, web, etc. The point is, the stuff that lives here is independent of any platform. The logic in domain is its heart, and it mainly consists of two things:

    Entities
    Use Cases

Entities are, to put it simply, a `POJO` class. They hold the least amount of logic, and their sole reason of existence is to shape the data that will be used by all the other applications, or rather whatever application sits on the top. For example, if we are building a notes taking application, we will have a `Note` entity which will look something like this:

```kotlin
data class Note(
val text: String,
val author: String
)
```

Notice how there is no `id` field, or any other `primary key` field, or any annotation processor used. There is a reason behind all of this madness. And that is dependency.
Adding annotations or mutating our entities so they can be used for persistence is not our goal here, especially in the domain layer. Adding 3rd party dependency will weaken the heart of your architecture, as now your domain depends on the 3rd party module. Purists who follow clean architecture to its true core goes as far as creating custom mappers for `JSON` to `String` and back to `JSON` even though there are fully fledged 3rd party libraries such as `GSON` by google Inc. to do that. Including `GSON` in entities, such as, would do not harm technically to your domain as the library is pretty stable and going nowhere. However, now you have a dependency in your domain, and in future if support to `GSON` ceases to exist or changes, you would have to go and make changes to your entities, hence breaking the second rule of `SOLID` principles, the `OPEN/CLOSED PRINCIPLE`.

Use cases are contracts that give us information about what sort of action will be taken with our entities. For example, following our note taking application, here what our use cases will look like:
```
IAddNoteUseCase
IDeleteNoteUseCase
IUpdateNoteUseCase
IGetNoteUseCase
```
Notice how each use case has a prefix `I` in the beginning. That tells you that these use cases are interfaces and not actual implementation. The reason is again pretty simple, dependency. Ultimately, our `Presenter`, which will live in the presentation layer, will use these use case classes. However, we don’t want our presenter to know how the use case are being implemented. Presenter should have no such knowledge, as it will never in its lifetime will try to change it. So why release this information in the first place. This is a good way to safeguard our implementations and this is what causes the presentation layer to be dependent on our domain layer and not the other way around. Even though run time dependency goes from `presentation to domain`, the source code dependency goes from `presentation to use case implementation to IUseCase`. In doing so, we  followed the dependency inversion principle.

This simple feature is what every architect use to spin those dependencies that travel from one module to another and turn them 180′. Also, note how we have separated every use case into a single unit, hence following single responsibility principle and interface segregation principle.

Use cases are typically composed of repositories which can help them pull all the models they need. Here, models means data that will be sent through us from various data sources. However, use cases will not be aware of these models, as the repositories will convert these models to entities and nicely serve it to our use case, which will simply propagate the result by wrapping it in a state object. Read `railway programming paradigm` to learn more about it.

Now let’s talk about the data layer.

# DATA  

Our Data layer usually consists of the following:
```
IRepositories
IDataSources
Models
```
Needless to say, again we are only releasing the contract to the domain layer. Domain layer should only know what can be done, instead of how it is being done. Our repositories will be cleanly segregated based on the feature. Usually, our repositories will be made up of our local data source implementation, our remote data source implementation, and our custom network info class, which can act as a utility class for checking the network state of the application that is being implied Again, we will use `INetworkInfo` as the only thing in our case, for instance, that we want to use is a helper method class,  which will return us a boolean based on the network state.

Data sources is where we will put all our low-level details. Feel free to make use of annotations in the models and to use data persistence throughout your application. Your logic is very cleanly placed in a single place which is the repository. You can decide the order in which persistence will occur. A general rule of thumb is to fetch the active remote data, save it in local storage, and send the data to the upper layers. In a situation where network state is inactive, you can write the logic that can make this local storage data kick in, while you are still checking the network state in the background with exponential back off.

Most of the work will be done by the data sources. The only thing we expect from our data sources is a model which should be non-null. Use all the tools at your disposal that your language provides you here. In cases where data sources fail to fetch a model, we can raise and `exception`. This raised `exception` will be neatly caught in our repository. What we expect from out repository is a `Result`. Now this `Result` can further have as many states as you want, but for simplicity, we can say that it is composed of `Result<model, exception>`, with `Result.Success` and `Result.error` as its child classes. This can be easily used in the use cases to see what kind of response is provided be the repository and based on an error or a failure we can propagate it further to our views.

> Read about “State Transition Machines” to understand more about the concept of states. 

> Read about “Sealed classes” or “Enums” to understand how we can create these state hierarchies.

Last, but not the least, comes the presentation layer.

# PRESENTATION
 
Our presentation layer will consist of the following:
```
Views
Presenters
```
If you have multiple presenters working together and assisting a single view, feel free to use interfaces as we did to protect your view having a transitive dependency on your presenters. To keep things simple in our case, we will go ahead and simply create the concrete classes that hold the logic. In simple applications, this approach is fine as the flow of control usually goes from `view to presenter` all the time. 

Here we have multiple choices as we are at the edge of our architecture. Anything we do can be and should be easily swappable as this does not concern our other layers. Being an android developer, I see the trend of migration from MVP to MVVM (Yes I know MVI is the ‘new boom’. However, technically all the MVs are not architectures). With our clean architecture implemented, this migration is painless as all we would need to do is copy the code lying in presenter to our view model and switch it inside the view. We can now reap the benefits a viewmodel offers us, without worrying about how our data sources or repositories will change. 

This ability to switch different components without touching others is what clean architecture provides us. Now in theory, it sounds like an overkill and some might argue a lot of this can be achieved without implementing such modules. And you are right. All of the above can be done without all the segregation we have talked about so far. However, in doing so you will create an architecture nevertheless particularly unique to this application, and the understanding and complexities of this architecture will be implicit to those who created it. An average lifespan of a software developer in a company is 1-2 years, whereas the software being written usually lasts 5-6 years. That means for those 3-4 years, the new developers will be in a constant state of confusion and disapproval. Every code will be questioned by them as they will fail to realize how things were done the way they look in code right now. On the other hand, if everything is properly put in its own module and the architect keeps our good old friend dependencies in check, the boundaries will make clear sense. Even the new developers will pick up the project quickly and the thought of not leaking information along these boundaries will be clear. On top of that, putting Test Driven Development (TDD) in the mix will further provide strong documentation about each and every module, as per to say what it can do and what it can’t. 

As stated in the book with the eponymous title, making software is scientific and not mathematic. We are constantly trying to prove what our software cannot do, and not what it can do. And anything we fail to prove that it cannot do for a long time, hence become one of the features our software which it can do.