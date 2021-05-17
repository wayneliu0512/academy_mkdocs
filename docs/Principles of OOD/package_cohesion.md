# Granularity
The class, while a very convenient unit for organizing small applications, is too ﬁnely grained to be used as an organizational unit for large applications. Something “larger” than a class is needed to help organize large applications. 

## Designing with Packages
In the UML, packages can be used as containers for a group of classes. By grouping classes into packages we can reason about the design at a higher level of abstraction. The relationships between those packages expresses the high level organization of the application. But this begs a large number of questions.

1. What are the best partitioning criteria?
2. What are the relationships that exist between packages, and what design principles govern their use?
3. Should packages be designed before classes (Top down)? Or should classes be designed before packages (Bottom up)? 
4. How are packages physically represented? In C++? In the development environment?
5. Once created, to what purpose will we put these packages?

To answer these questions, I have put together several design principles which govern the creation, interrelationship, and use of packages.

## The Reuse/Release Equivalence Principle (REP).
*"THE GRANULE OF REUSE IS THE GRANULE OF RELEASE. ONLY COMPONENTS THAT ARE RELEASED THROUGH A TRACKING SYSTEM CAN BE EFFECTIVELY REUSED. THIS GRANULE IS THE PACKAGE."*

## The Common Reuse Principle (CRP)
*"THE CLASSES IN A PACKAGE ARE REUSED TOGETHER. IF YOU REUSE ONE OF THE CLASSES IN A PACKAGE, YOU REUSE THEM ALL."*

Classes are seldom reused in isolation. Generally reusable classes collaborate with other classes that are part of the reusable abstraction. The CRP states that these classes belong together in the same package.

A simple example might be a container class and its associated iterators. These classes are reused together because they are tightly coupled to each other. Thus they ought to be in the same package.

it is common for packages to have physical representations as shared libraries or DLLs. If a DLL is released because of a change to a class that I don’t care about, I still have to redistribute that new DLL and revalidate that the application works with it.

Thus, I want to make sure that when I depend upon a package, I depend upon every class in that package. Otherwise I will be revalidating and redistributing more than is necessary, and wasting lots of effort.

## The Common Closure Principle (CCP)
*"THE CLASSES IN A PACKAGE SHOULD BE CLOSED TOGETHER AGAINST THE SAME KINDS OF CHANGES. A CHANGE THAT AFFECTS A PACKAGE AFFECTS ALL THE CLASSES IN THAT PACKAGE."*

If the code in an application must change, where would you like those changes to occur: all in one package, or distributed through many packages? It seems clear that we would rather see the changes focused into a single package rather than have to dig through a whole bunch of packages and change them all. That way we need only release the one changed package. Other packages that don’t depend upon the changed package do not need to be revalidated or rereleased.

The CCP ampliﬁes this by grouping together classes which cannot be closed against certain types of changes into the same packages. Thus, when a change in requirements comes along; that change has a good chance of being restricted to a minimal number of packages.


