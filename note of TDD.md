TDD

Experience told us, if you write codes only from the functions, you'll miss a lot of kinds of errors that may happen. The more you write, the more you tend to ignore the errors, sometimes even on purpose. TDD provides a way to start from "faliures".
To start implementing a features, first design the "acceptance test", which is the correct path of this feature, that may face some failure. Then, refactor the code to make it pass. Design another test and make it pass, on and on.

- Levels of Testing -
-- Acceptence test: It could be also called "functional tests", "customer tests". The point is to make the tests end-to-end. It's to let the code execute as expected, without changing the existing features (that's why we call it refactor).
-- Integration test: It's to check if our code could work well with other's code, which is unchangable. It could be code from other team, could be third-party library, could be open source project...

II. Test-Driven Development With Objects
- values and objects
objects are different, even if they have exactly the same contents and states. They can diverge later. It's different with values, which are immutable.

- communication patterns
A communication pattern sets up a series of rules about how objects communicate with each other.  

- mock objects
some dependencies you have a specified input and output. Instead of using them, use a mockup.

Tools
- test fixture: a fixed state that exists at the start of a test. @Before could be a fixture. @After could tear down the fixture.
- assertThat(T actual, Matcher<? super T> matcher)
Matcher is an interface checking if a condition is satisfied.
- Mockery
Encapsulate an object or an class. This object will be treated as a black box of the class.

Maintaining the test-driven cycle
An past acceptance test shouldn't fail again.
Start from simplest "success" case, to get some achievement. 
When you order dishes at a restaurant, you won't order the recipe with the ingredients. 
In real life, third parties don't just give you an array or a value. The resources could be a service, could be a thread. This means not all third-parties can be mocked.

- kinds of peer objects
1. Dependencies: services needed to implement the functions.
2. Notifications: when I change, I need to let peers know my updates.
3. Adjustments: peers that adjust the object's behavior. It's not just an dependency.

It's better that an object doesn't need to have the knowledge of the bigger environment.
-- TDD helps design:
1. describes a "what" before "how"
2. tells how to split the code into components
3. helps know about dependencies better

* An interface describes whether two components fit together, while a protocol describes whether they work together.
We want interfaces to be as narrow as possible, even though we need more interfaces.

-- Organize the code into two layers:
1. implementation layer: a graph of objects about how objects respond to events. Describe "how".
2. declarative layer: describe each fragment. Describe "what".

- Build on Third Party Code
We don't want to read, change code from other's, and we can't mock other's code. There should be a adapt layer to deliver the output and the input, as a proxy.
One thing to mock the third party code is exception. This is not usual in normal cases.

- Auction Sniper

--The auction protocol:
Bidder's commands:
Join: a bidder joins an auction
Bid: a bidder sends a bidding price
Auction's events:
Price: the currently accepted price, min increment the next bid must raised by, the name of bidder who bid the price
Close: annouce the close.

Unlike unit tests, TDD allows some failures.