# microservice-tutorial: Stage 1

In this stage, we'll talk a bit about what a microservice is and how it's used in software engineering. We won't be actually writing any code in this part of the tutorial, this stage purely conceptual.

## What's a mircoservice?

A microservice is a small, focused, and maintainable unit of a system. A system that is designed using microservices has high "cohesion" but low "coupling". When systems are designed in this way, the major benefit is ability to maintain, debug, deploy and develop individual components with a relatively low cost as opposed to a more coupled system.

### Cohesion

Cohesion is a measure of how well related logic is kept in one place. If related logic is spread all over a system, the system can suffer performance impacts of communication across logical component boundaries. Often this means that related state is duplicated everywhere the related logic exists. When we go to make changes to how the logic works, we might end up "cascading" the effects of the changes across our system. 

This is especially concerning in systems that use a shared component in many unrelated places. When that's the case, it's difficult to make changes to the behavior of the shared component. If a certain application of the component necessitates a change in the interface of the component, you have to be careful that this interface change is compatible with other uses!

Let's use an example to demonstrate - you are working on a project with 4 other people, except the people on your team are spread across the globe. Your communication suffers because it takes a full day or longer for your teammates to reply to your emails due to the time difference. If your team agreed to equally divide up the project so that everyone gives input on every part of the project, your team will have poor cohesion. It seems counter-intuitive, since the team is doing *everything* together, but that model also requires a lot of communication and real time collaboration which is highly expensive when everyone has to work remotely.

To make the team more cohesive, it actually makes more sense to split project into individual subproblems that can be solved more or less independently. Then, your team could divide up the subproblems. Since the sub-problems don't require a lot of collaboration to solve, your team can reduce the expensive cross-globe communication to when it's really necessary. Similarly, in software systems it makes more sense for components to be as self sufficient and singularly focused as possible - solving a focused subproblem in one component makes the system overall more efficient and modular.

### Coupling

Coupling is a measure of how much components depend on each other. Some coupling is inevitable between components of a system, since separating out functionality into smaller components requires these components to work together for the system to function. Microservices aim to reduce coupling between system components as much as possible by taking all the parts of the system that have high cohesion (see the section above) and bundling them together. The resulting components will depend less on each other. While these interfaces can't be completely elminated, reducing them as much as possible makes a system much easier to maintain.

It's useful to have low coupling between components since it makes it easier to replace the part of the system with a different implementation or process without requiring complementing changes in other components. In highly coupled systems it can be difficult to even change a component since components can depend on each other in strange ways. Fixing a bug can then require changes in multiple parts of the system.

Let's use a warehouse analogy to demonstrate coupling. Let's say two employees need to load a truck. One employee (selector) decides the highest priority packages to load in the truck, retrives them, and then hands it to the second employee (loader). The loader is responsible for loading the package into the truck in a way that consumes the least amount of space. The two employee system has high coupling since the selector can't select the next package until the loader takes it off her hands, and the loader can't load more packages until the selector hands it to her. If we were to replace either employee with a high-tech robot arm, we would then have to train that employee to work with the robot which can be more expensive than we can afford. 

To fix the coupling, we can add a conveyor belt to the system. The selector places packages on the conveyer belt, and the loader picks them off to load them. In this scenario, the loader and selector don't need to pay attention to each other, they can both pick up/place packages at will. An additional benefit of this solution is that we can now replace one or both employees with a robot arm without having to train the other employee, since their workflow won't be impacted.

### Coupling & Cohesion in Microservices

A microservice-oriented architecture allows a software system to attain low coupling and high cohesion. This means that components are better at performing one focused function, and don't rely heavily on other systems to do their work. Moreover, when we decide to change something about a microservice - or replace it entirely - we don't need to make sweeping changes across the system since the related logic and functionality is scoped to the component itself.

Proper application of microservices is very similar to the core computer science concepts of encapsulation and abstraction. Both of the mechanisms allow us to create systems that are inexpensive to build and maintain, and that are flexible enough to evolve and improve over time as the needs of your users change.

## When should I use microservices?

Just because we've been talking about all the benefits of microservices doesn't mean that they are always the right tool for the job. Microservices can be expensive to build and maintain, since deploying several fragmented components can often introduce a lot of unwanted overhead. For example, in cloud computing, each microservice typically represents an individual web server. Each microservice will then require it's own deployment pipeline, need to be to be tested separately, and might need to repeat a lot of common functionality like boiler-plate to authenticate a user session.

### A case for the Monolith

Often, microservices aren't the starting point for a software organization. Many large software companies today started out with a single application that handled all functions, which are typically referred to as a **Monolith**. Monoliths will often have bad code cohesion and a high amount of coupling - these properties will make it harder and harder to make changes over time as more functionality is added, since small changes have the potential for widespread impact in behavior. 

At the same time, Monolithic applications side-step a lot of the overhead issues related to microservices. With one application, you only need to deploy one application before your users can realize the functionality of your software. There's typically one (or a few) codebases to maintain, and a lot of common functionality can be shared within the application leading to less repeated code. As a result, it's often the case that starting with a monolithic app is a **good** move for an engineering organization. It helps engineers work faster to deliver more features without taking on the overhead of microservices.

### Scale with microservices

When would we make the transition over to microservices? This really depends from project to project, but a good indication is when coupling within the system erodes cohesion to the point where it starts impact feature development. This slows down the delivery of features that users find valuable, as engineers struggle with what's called **technical debt**. It's the cost of adding coupling and decreasing cohesion in order to deliver features quickly. When it piles up, it prevents the codebase from scaling.

**Scaling** is somewhat of a buzzword in the software world, but it is a core part of software engineering, since we're always thinking about how our systems perform as we add more users, more data, and more functionality into the mix. More users will naturally result in a larger volume of transactions in a system. More data requires computing resources to both compute on and store the data. More functionalities make the system ever more complicated. A combination of these factors along with a monolithic code base cause real problems. 

The application might function correctly for the first 1,000 users supported on the system, but at 10,000, the system slows to the point where it's unusable. Some stragies like multiple instances of the application can help split traffic, but ultimately this extremely expensive - linearly scaling the number of computing resources with the number of users or data your system can handle is unsustainable, especially since the added cost (financial or otherwise) doesn't ensure greater profits. Microservices can help this scenario by splitting up traffic and logic in a way that scales properly.

To understand how scaling works let's visualize an example - Imagine a restaurant that serves 10 different items. 9 of these items are quick and easy to produce, but 1 of these items are expensive. For all 10 items, the same groceries are bought, the same kitchen is used, and the same staff works on them. However that 1 item is hard to do right, takes a lot of space in the kitchen and requires multiple cooks to prepare. Assuming each of the items is equally popular, the restaurant can only serve a certain amount of customers until they simply can't keep up with orders - there's not enough space, dishes or staff to cook every order. As a result the restaurant can't serve more customers, and therefore can't make more profits.

To help this restaurant scale, one thing the owner can do is to split off that 1 expensive item into it's very own food truck. With this food truck focusing on just this one item, it's much easier to plan and prepare that item. The restaurant can then drop that item of the menu, and continue to serve the 9 cheaper items. When this is done, a lot of resources get freeded up in the kitchen and the restaurant is suddenly able to handle way more orders, which means more customers and more profits! Meanwhile loyal patrons can still get the expensive item from the foodtruck, which can specially configured to produce that one item.

Similarly, sometimes in software it makes sense to split large applications into focused microservices that are specialized to specific task at hand. Microservices can lead the operational benefits within software organizations as well, since these organizations can naturally split up into teams dedicated to a set of microservices. This is a great way to scale the number of *engineers* working on the project as well!

## Further Reading

For the purposes of this tutorial, the above will be enough context about microservices to understand how build them and when to use them. If you'd like to do further reading I recommend picking up copy of [*Building Microservices* by Sam Newwan, Published by O'Reily Media Inc.](https://www.amazon.com/Building-Microservices-Designing-Fine-Grained-Systems/dp/1491950358) from which the above content was also partly adopted. 
