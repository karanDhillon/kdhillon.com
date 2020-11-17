---
layout: post
title:  "The morning after syndrome"
---

Anyone who has worked on a mid to large sized project, has experienced the phenomena when the changes you made for the day are not present the next day when you touch the codebase as your work got overwritten by someone else. This is what know as `“The Morning After Syndrome”`. Today we will do the RCA for this and try to understand how we can solve this issue in our projects.

> Root Cause – Presence of a Cyclic Dependency Graph in your components.

![Cyclic dependency graph]({{site.baseurl}}/assets/images/Cyclic-dependency-graph.png)

In the above diagram, you will notice that are our components are well laid and the dependencies flow from top to bottom. A change in the authorizer will not cause a change trigger in the database for example. The only component that needs to be rebuild are those that are directly dependent on our authorizer component, which in our case is the main component. This is what is called a `“Directed Acyclic Graph”` or `“DAG”` for short. 

Having your components laid out where they can be separately build and deployed is what an architect wants to achieve. This has several benefits as follows:
* Each component’s testing becomes easier and practical.
* Teams will be able to migrate to new releases of that component as it is not strongly coupled with other components and most importantly.
* Teams can work independently without overwriting each others work. 

This utopia, however can be seriously taken away by adding one simple dependency in the mix to our architecture, having ,for example, our entities depend on our authorizer.

If a developer unknowingly creates a entity to be dependent on one of the classes in the authorizer component in order to not “duplicate” the code, we will have more problems than what this idea tried to solve in the first place. To realize the extent of damage done, let us imagine a scenario where our authorizer component had some new changes and needs to be redeployed just like we did in the beginning of this blogpost. However, this time there are lots of actors involved:
* Entities and main components will be rebundled because of their direct dependencies on authorizer component.
* Database and Interactor components will be rebundled because the entity component has changed. There could be a scenario where entity component is not even using this new code that was published in the authorizer component, however, it will still be rebuilt because of its direct dependency on authorizer.

Notice how a single dependency has completely thwarted the architect’s effort to maintain their plugin style architecture. The developer’s original intent might be to use Permission class from the authorizer component, which is nothing than a Util class. However, in doing so, an entire component gets depended on another. The solution will be discussed later, but in our case this could have been simply avoided if the developer recreates the Permission util class in the entities package as well. 

> Read Sandi Metz's [Wrong abstraction](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction) article for further reference.

# Solution
#### Apply the Dependency Inversion Principle

![Soultion 1]({{site.baseurl}}/assets/images/Solution-1.png)

Our primary goal is to convert our cyclic graph back to DAG. In order to do that we have to resolve this new `entities to authorizer` dependency. One solution is to apply dependency inversion principle. We can introduce a `IPermission` interface which will be part of entities component. Whichever entity class was previously directly referencing the permissions util class can now do so by using our new `IPermissions` interface. This will protect our entities component from knowing too much detail about how Permissions class is implemented in the authorizer component. There will also be no need to rebundle the entities component this time as there is no direct dependency.

#### Create a new component that both the entities and authorizer depends upon

![Soultion 2]({{site.baseurl}}/assets/images/Solution-2.png)

In this solution, we simply move all the classes that are involved in entities and authorizer components into a new component. What we are doing here is simply separating what frequently changes from the part which doesnot. In terms of design patterns, we just applied the strategy pattern.

Spawning new components is not the best approach as the new component might not make great sense in the grand scheme. Purists can argue over how it could lead to the violation of the Common Closure Principle – Classes that change together should be in the same component. However, applying this solution does give us our DAG back and gets rid of the cyclic nature our system was heading towards. 

It would make sense to take out some time at this point and explore what your components are composed of. You want to enjoy separation of concerns as long as the separations you make, make good sense in the long run. A good thought to keep in mind at this point would be how you will version your components for new releases. Too many versioning would allow slow adoption whereas less components would cause less reusability.  This is also known as “Cohesion principles tension diagram”.

In the end, each architect has to decide which benefits they want to enjoy at the current moment. For now, reusability might not make much sense and coming out with new versions would. Later, reusing would more make sense than better adoption rate. What one does will depend on which state the architect wants the system to be.