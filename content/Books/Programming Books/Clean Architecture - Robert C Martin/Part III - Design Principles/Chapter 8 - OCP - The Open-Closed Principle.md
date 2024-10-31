---
id: 01JBCFN1AF2FJ9R3HAXJ80K7AC
modified: 2024-10-29T12:07:50-04:00
title: Chapter 8 - OCP - The Open-Closed Principle
tags:
  - srp
  - design-systems
  - architecture
  - books
  - programming
---
## 8  
OCP: THE OPEN-CLOSED PRINCIPLE

![Image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780134494272/files/graphics/CH-UN08.jpg)

The Open-Closed Principle (OCP) was coined in 1988 by Bertrand Meyer.[1](https://learning.oreilly.com/library/view/clean-architecture-a/9780134494272/ch8.xhtml#ch8fn1) It says:

_A software artifact should be open for extension but closed for modification._

In other words, the behavior of a software artifact ought to be extendible, without having to modify that artifact.

This, of course, is the most fundamental reason that we study software architecture. Clearly, if simple extensions to the requirements force massive changes to the software, then the architects of that software system have engaged in a spectacular failure.

Most students of software design recognize the OCP as a principle that guides them in the design of classes and modules. But the principle takes on even greater significance when we consider the level of architectural components.

A thought experiment will make this clear.

### A THOUGHT EXPERIMENT

Imagine, for a moment, that we have a system that displays a financial summary on a web page. The data on the page is scrollable, and negative numbers are rendered in red.

Now imagine that the stakeholders ask that this same information be turned into a report to be printed on a black-and-white printer. The report should be properly paginated, with appropriate page headers, page footers, and column labels. Negative numbers should be surrounded by parentheses.

Clearly, some new code must be written. But how much old code will have to change?

A good software architecture would reduce the amount of changed code to the barest minimum. Ideally, zero.

How? By properly separating the things that change for different reasons (the Single Responsibility Principle), and then organizing the dependencies between those things properly (the Dependency Inversion Principle).

By applying the SRP, we might come up with the data-flow view shown in [Figure 8.1](https://learning.oreilly.com/library/view/clean-architecture-a/9780134494272/ch8.xhtml#ch8fig1). Some analysis procedure inspects the financial data and produces reportable data, which is then formatted appropriately by the two reporter processes.

![Image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780134494272/files/graphics/08fig01.jpg)

**Figure 8.1** Applying the SRP

The essential insight here is that generating the report involves two separate responsibilities: the calculation of the reported data, and the presentation of that data into a web- and printer-friendly form.

Having made this separation, we need to organize the source code dependencies to ensure that changes to one of those responsibilities do not cause changes in the other. Also, the new organization should ensure that the behavior can be extended without undo modification.

We accomplish this by partitioning the processes into classes, and separating those classes into components, as shown by the double lines in the diagram in [Figure 8.2](https://learning.oreilly.com/library/view/clean-architecture-a/9780134494272/ch8.xhtml#ch8fig2). In this figure, the component at the upper left is the _Controller_. At the upper right, we have the _Interactor_. At the lower right, there is the _Database_. Finally, at the lower left, there are four components that represent the _Presenters_ and the _Views_.

![Image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780134494272/files/graphics/08fig02.jpg)

**Figure 8.2** Partitioning the processes into classes and separating the classes into components

Classes marked with `<I>` are interfaces; those marked with `<DS>` are data structures. Open arrowheads are _using_ relationships. Closed arrowheads are _implements_ or _inheritance_ relationships.

The first thing to notice is that all the dependencies are _source code_ dependencies. An arrow pointing from class A to class B means that the source code of class A mentions the name of class B, but class B mentions nothing about class A. Thus, in [Figure 8.2](https://learning.oreilly.com/library/view/clean-architecture-a/9780134494272/ch8.xhtml#ch8fig2), `FinancialDataMapper` knows about `FinancialDataGateway` through an _implements_ relationship, but `FinancialDataGateway` knows nothing at all about `FinancialDataMapper`.

The next thing to notice is that each double line is crossed _in one direction only_. This means that all component relationships are unidirectional, as shown in the component graph in [Figure 8.3](https://learning.oreilly.com/library/view/clean-architecture-a/9780134494272/ch8.xhtml#ch8fig3). These arrows point toward the components that we want to protect from change.

![Image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780134494272/files/graphics/08fig03.jpg)

**Figure 8.3** The component relationships are unidirectional

Let me say that again: If component A should be protected from changes in component B, then component B should depend on component A.

We want to protect the _Controller_ from changes in the _Presenters_. We want to protect the _Presenters_ from changes in the _Views_. We want to protect the _Interactor_ from changes in—well, _anything_.

The _Interactor_ is in the position that best conforms to the OCP. Changes to the _Database_, or the _Controller_, or the _Presenters_, or the _Views_, will have no impact on the _Interactor_.

Why should the _Interactor_ hold such a privileged position? Because it contains the business rules. The _Interactor_ contains the highest-level policies of the application. All the other components are dealing with peripheral concerns. The _Interactor_ deals with the central concern.

Even though the _Controller_ is peripheral to the _Interactor_, it is nevertheless central to the _Presenters_ and _Views_. And while the _Presenters_ might be peripheral to the _Controller_, they are central to the _Views_.

Notice how this creates a hierarchy of protection based on the notion of “level.” _Interactors_ are the highest-level concept, so they are the most protected. _Views_ are among the lowest-level concepts, so they are the least protected. _Presenters_ are higher level than _Views_, but lower level than the _Controller_ or the _Interactor_.

This is how the OCP works at the architectural level. Architects separate functionality based on how, why, and when it changes, and then organize that separated functionality into a hierarchy of components. Higher-level components in that hierarchy are protected from the changes made to lower-level components.

### DIRECTIONAL CONTROL

If you recoiled in horror from the class design shown earlier, look again. Much of the complexity in that diagram was intended to make sure that the dependencies between the components pointed in the correct direction.

For example, the `FinancialDataGateway` interface between the `FinancialReportGenerator` and the `FinancialDataMapper` exists to invert the dependency that would otherwise have pointed from the _Interactor_ component to the _Database_ component. The same is true of the `FinancialReportPresenter` interface, and the two _View_ interfaces.

### INFORMATION HIDING

The `FinancialReportRequester` interface serves a different purpose. It is there to protect the `FinancialReportController` from knowing too much about the internals of the _Interactor_. If that interface were not there, then the _Controller_ would have transitive dependencies on the `FinancialEntities`.

Transitive dependencies are a violation of the general principle that software entities should not depend on things they don’t directly use. We’ll encounter that principle again when we talk about the Interface Segregation Principle and the Common Reuse Principle.

So, even though our first priority is to protect the _Interactor_ from changes to the _Controller_, we also want to protect the _Controller_ from changes to the _Interactor_ by hiding the internals of the _Interactor_.

### CONCLUSION

The OCP is one of the driving forces behind the architecture of systems. The goal is to make the system easy to extend without incurring a high impact of change. This goal is accomplished by partitioning the system into components, and arranging those components into a dependency hierarchy that protects higher-level components from changes in lower-level components.

[1](https://learning.oreilly.com/library/view/clean-architecture-a/9780134494272/ch8.xhtml#ch8fn-1). Bertrand Meyer. _Object Oriented Software Construction_, Prentice Hall, 1988, p. 23.