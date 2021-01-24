creational, structural, behavioural
Creational patterns concern the process of object creation. Structural patterns deal with the composition of classes or objects. Behavioral patterns characterize the ways in which classes or objects interact and distribute responsibility.

# Creational patterns
## Abstract Factory
When:
1. a system doesn't care about how the objects are created
2. a system configured with families of objects
3. a system needs to use an enforced set of products
4. a system only showing the interfaces not the implementations
With Abstract Factory, client won't know the class names in the server.

## Builder
Abstract Factory means to solve different kinds of factories, builder means to solve different parts of a product.
When:
1. a complex object has an independent algorithm
2. the construction process must allow different representations for the object
that's constructed. ???
Usage:
Builder as an interface, concrete builders implements it. Director organizes(uses) the builders. Director is a part of the product.
Concrete builders can be reused by other builders. 
* Step by step

## Factory "Method"
See, the point is "method", meaning createBla(); so this can be inside the abstract factory.
When:
1. a class doesn't know the concrete classes it need to create objects
2. a class needs it subclasses to determine the concrete classes
3. a class wants to delegate a helper subclass, and it's needed to know which subclass is the delegate ???
Factory is often connected to abstract factory
Usage:
Kind A is using Kind B, kind A has subclasses A1, A2, responsively using B1 and B2. Then A and B both be abstract classes or interfaces.
* eg
Human "wears" clothes.
Man extends Human, Woman extends Human.
Jeans extends Clothes, Skirt extends Clothes.
Man "wears" Jeans and Woman "wears" Clothes. Imagine wear as an object creator.
* ge
Variation: parametered factory method, create(String object). Either with switch case, or with reflection. It's still good to use the ancestors rather than concrete creators.

## Prototype
Sometimes, if subclasses have very similar structures, only the kind varies, it's a a waste to create lots of subclasses. Then, there could be a prototype class, to make copies of different kinds of objects, with different parameter of kind.
The main part of prototype is "clone()".
When:
1. classes are specified at runtime (when I read this parameter, I know this is what kind)
2. avoid building a complex hierarchy of factories
3. 
* deep copy means clone(), not just a copy of reference, sometimes with Serialization. So serialization is meant for objects, not for "class".
The difference is the usage between prototype and abstract factory. Using a prototype, means you only have one kind of product, though the ingredients may differ. Using abstract factory, means you have different kinds of products, and the ingredients are fixed. But inside the prototype, the ingredients are high classes, which means the ingredients can be more concrete.

## Singleton
When:
1. only one instance needed
2. extend the one object without changing the 
Singleton is an enhanced global variable. 
1. lazy initialization with thread safety
   ```
   public class Singleton {
	   private static Singleton singleton;
	   private Singleton() {}
	   public Singleton getInstance() {
		   if(singleton == null) {
			   synchronized (Singleton.class) {
				   // double check
				   if(singleton == null) singleton = new Singleton();
			   }
		   }
		   return singleton;
	   }
   }
   ```

# Structural Patterns
## Adapter Pattern
When:
1. the existing class's methods doesn't match the ones we need
2. create a class with unrelated or "unforeseen" classes, who don't share compatible interfaces
3. can't adapt some classes' methods by inheriting

## Bridge Pattern
Imagin an interface, it has 3 subclasses implementing it, say 3 different systems. Then if there are some specific demand of this interface, there would be 3 subclasses inheriting these 3 2nd layers, means there will be 1 interface + 6 subclasses. Using bridge, there could be an interface standing in the same layer as the interface, and let the 3 subclasses inherit these 3 subclasses. Then, there are 2 interfaces, 6 subclasses. Seems not making sense? No, there are only 2 layers. This stops from exponential explode. Say there are multiple subsystems under the systems, it would explode.
When:
1. avoid building a permanent inheritance between an interface with a concrete class -> like when it needed to decide in runtime
2. you want the "root" abstraction be very extensible
3. change in the "root" abstraction don't affect subclasses
4. hide the implementation of abstraction (abstract class only, interface doesn't make sense)
5. avoid class explosion
6. hide the client that objects are sharing one implementation
let a double do the shts for you, then a bridge lies between you. 
Abstraction imp-----------> Implementor
	|							|
subclasses 				concreteImplementor
all the classes in the left side needs to implement something, let's say EImp(); different subclasses may use different implementations. Say subA has a AImp(), subB has a BImp(). If AImp() is plain written in subA, it's alright, then it's transparent. So, if you want to hide the implementation of AImp, use a class to encapsulate it. Since there are different kinds of Imps, then make an abstraction -> Imp(), and let AImp, BImp inherit this abstraction. 

## Composite Pattern
When:
1. Part-whole hierarchies of objects
2. eliminate the difference between objects and compositions
Add and remove functions are essential, but does not always make sense, in leaf. Throw exception in Add and Remove in the leaf composite.

## Decoration Pattern
Figure out: what's a decorator? Is it a "partial" thing added to an interface, or an interface including the "partial" thing? --------LATTER. A "BIGGER" THING.
When:
1. add responsibilities, while it's still the original interface
2. responsibilities can be withdrawn ???
3. impractical to subclass ---> you are trying to build an abstract, with decorator
* better to be lightweighted --> focus on one interface

## Facade Pattern
When:
1. use one interface to operate a complex system, including subsystems
2. decouple clients from subsystems, implementations of abstracts
3. give each layer of subsystem an entry, to communicate with other system solely

## Flyweight Pattern
Some objects are shared frequently by different kinds of objects. The content in flyweight is fixed, but the objects using it differ. Flyweight can be used in different way, but they are serving some definite purpose. This helps save space of duplicate objects with the same content.
When:
1. the application needs lots of objects
2. then the cost of storage of objects is very high
3. most object state can be extrinsic
4. if the extrinsic is removed, the objects can be shared and replaced
5. the objects don't need identity, they are hot plugin
There is a flyweight pool storing flyweights with different intrinsic state, then there is a structure looking up flyweight by state.
Problem: if we want to use a concrete flyweight, are we using the original one or a copy?
Answer: Same reference at any usage, because we are using map or table.
ConcreteFlyweight: the flyweight with fixed intrisic state
UnsharedConcreteFlyweight: doesn't supporting being shared, has a container of some concreteFlyweight
Clients shouldn't create or get flyweight themselves. They need to use a FlyweightFactory to get what they need. There's a table-like structure to store key-flyweight pairs.

## Proxy Pattern
Use a temporary object in place of a slow-loading object. The proxy shares the same hierarchy as the oringinal one, their structures look all the same. The proxy lives until the original one shows up. Stands in between the client and the real object. Must contain the real object's reference. They have exactly the same input, interface, and can be used as the real object, just without content.
Categories:
1. remote proxy: access objects from other server, then we have a local proxy
2. virtual proxy: an expensive object to create like a big image, we have a fake one with the same shell
3. protection proxy: the real objects have different permission sets
4. smart reference: to check the state of some objects being accessed
If the proxy is to handle some abstracts, the proxy doesn't need to know the specific implementation of subclasses. If the proxy is to create a real object, it does need.

Adapter uses existing interface, since it needs to accomadate it. Facade makes a new interface to client, for it provides new methods and usages.

# Behavioral Patterns

## Chain of Responsibilities

One request has one receiver. If we don't want to hard code the receiver, we send it to any receiver, if it can't handle, let it pass it to the next receiver, until the right receiver receives it. 
Then, the sender won't need to know who to receive it.
When:
1. more than one receiver can handle an ambigious request. The receiver needs to be asserted automatically
2. send a request avoiding naming the receiver
3. request handled dynamically
used to pass the request to the "parent" in a composite.

## Command
An application has many different components, each of them has different function. If the application is very adaptable app, like SOTI's drag and drop, the functions are set by developers. Then we let all of the components do the same thing: Command. Developers need to implement what kind of command. Command is an abstract, subclass defines the kind of command.
An point I've been thinking: Earphones, different earphones can be used on different brands of phones (iphone excluded). If I push a button, it may play next, pause, turn up the volumn... how does a phone capture this? Command pattern can be used. There could be a unified standard interface for all earphones, they all use a "Command" object to encapsulate an operation and pass it to the phone, then the phone analyzes it. 
When:
1. make an action a parameter in an object
2. a command object can have an independant lifetime, when the request is sent, the object can be used at any other time
3. support undo. If there is an unexecute method in the Command object and it stores the original status, then the unexecute can return to a previous state. 
4. support logging changes	
5. encapsulate actions into transactions

## Interpreter Pattern
About grammar and analyzes sentences.
Please go though it later.

## Mediator Pattern
Within a system, there are lots of components, any pair would have some specific interaction. It's unpractical to let them know each other, for it would increase the coherance. Mediator can handle the interactions between objects. There could be an abstract called Mediator. Then subclasses can be specific interaction. 传话儿的。
Mediator: "If you are a component, mind your own business. Just tell me what you did, what you changed. I'll let the ones who need to know know."
Q: One component has one mediator for one interaction? 
A: Sounds reasonable. One mediator contains 2 or more specific objects. Control very specifically. And one object should store a mediator for one kind of interaction.
Avoid subclassing

## Memento Pattern
create a snapshot of previous state. convenient to revert. The Memento both expose part of internal state and avoid exposing all states.
One class to store state, one to be Mememto. The object stores the Mememto storing the previous state. UnExecute just calls setState(oldState)

## Observer Pattern
Mediator has three parties, one object change, one object transmit, one make actions. Observer has 2 parties, one to change, one to update. The changing object needs to store the observers dependant on its states, when change, tell them all. Observer as an interface, only. Therefore, storing observers by themselves, could be very expensive. Better to be references. Java makes it.
* a wonderful usage: to let others know I'm going to be deleted
Observer pattern makes subjct control the time to notify others. 
push: I tell you every thing I've changed
pull: I tell you I made some changes, you ask me what change you need after.

## Strategy Pattern
A pattern when concerns about algorithms. 
When:
1. related classes with different behaviors. RELATED
2. need a variant of an algorithm
3. encapsulate the algorithm, avoiding exposing internal structure
4. a better way to replace switch-case
a family of algorithms can be at the same hierarchy or have multiple hierarchies

## Template Method