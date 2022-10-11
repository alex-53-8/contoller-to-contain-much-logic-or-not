# Handler - why small implementation is good?
Controller, mouse event listner, message consumer and similar entities are types of a handler. A handler simply
accepts input, optionally makes a decision if possible to proceed with received input data, converts input
to a suitable format, and calls an underlying procedure.

The main question here is - how large can be a handler, what kind of logic a handler could contain?

Let's describe what a handler can be and do.

## A software application and a handler
Software application is computer program designed to carry specific task. A user/client can communicate with 
an application via a user interface or via network calls, like REST, message queue, etc. 
So, it means each application has a layer which is responsible for accepting user's/client's input,
verifying input, calling a specific procedure inside an application, and finally return output. 
Usually that layer is called a handler. Handler can be a REST controller with a set of endpoints, 
a keyboard or mouse event listener, a consumer for a message queue, a command-line arguments' parser and so on.

The main role of a handler is to accept and validate input data, prepare data from input for calling a procedure, 
make a decision if possible to proceed, call the procedure, and return result of procedure invocation if any.

So, the role of a handler is to provide a way to communicate with an application, which usually consists of multiple libraries.
Technically, it should not be difficult to provide to a user/client different interfaces to communicate with an application, but in different ways.

It should be quite similar, if an application exposes REST or SOAP, Protobuf interface or anything similar to call the same procedure.
Corresponding handler just accepts input data, prepares required input for a procedure, and calls a procedure.

```
| Handlers                             | Procedures               |
+--------------------------------------+--------------------------+
|                                      |                          |
+----------+                           |                          |
| REST     |-------+                   |                          |
+----------+       |                   |                          |
| Thrift   |-----+ |                   |                          |
+----------+     | |                   |                          |
| Protobuf |-----+-+------<-------->---| application's procedure  |
+----------+     | |     output/input  |                          |
|   ....   |-----+ |                   |                          |
+----------+       |                   |                          |
| RMI      |-------+                   |                          |
+----------+                           |                          |
|                                      |                          |
+--------------------------------------+--------------------------+
| Other Handlers                       | Other procedures         |
+--------------------------------------+--------------------------+
```

## How much logic should contain a handler?
To answer that question we would consider few aspects:
- Single responsibility principle from OOP design
- Testability

Probably even the second aspect is already redundant if consider only "Single responsibility principle".
The principle is described as following at [oodesign.com](https://www.oodesign.com/single-responsibility-principle)

> A responsibility is considered to be one reason to change. This principle states that if we have 2 reasons to change for a class, 
> we have to split the functionality in two classes.

It means if a handler contains validating input, validating authentication, preparing other parameters for a procedure call,
converting output, then such implementation clearly violates of a "Single responsibility principle". 
It is much better to move out logic, which does not belong to a handler explicitly, to abstractions or input processing chains.

From prospective of testing a handler that has long and complicated implementation with many dependencies is a nightmare :)
Using Test-driven development and keeping in mind "Single responsibility" principle helps to create small enough implementation 
which is easy to test and comply to OOP design patterns.

Very good example of such architecture is Spring Boot - validation can be done by providing special annotation on DTO fields, 
filters handle an authentication and assign all the parameters required by a context to a request attributes which can be easily used in a handler. Validation rules are subject of unit tests and can be tested separately, but definitely we should check if desired validation rules are applied on a handler level.

A handler becomes easy to test - all what we need to test is how a controller accepts input data, fails/succeeds on validation, and if output complies expected format. The rest of the things are tested separately by unit tests and finally by integration tests.

## Conclusion
Keep handler's implementation small, keep in mind OOP design principles, and optionally try to use Test-driven development as it facilitates
keeping an implementation small and easy to test.
