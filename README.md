# Controller - to contain much logic or not


## A software application and a controller
Software application is computer program designed to carry specific task. A user/client can communicate with 
an application via a user interface or an exposed network interface, like REST, message queue or anything similar. 
So, it means each application has a layer which is responsible for accepting user's/client's input,
verifying input, calling a specific procedure inside an application, and finally return output. 
Usually that layer is called a controller. Controller can be a REST controller with a set of endpoints, 
a keyboard or mouse event listener, a consumer for a message queue, a CLI parser and so on.

The main role of a controller is to accept input data, validate input, prepare data from input for calling a procedure, 
call the procedure, and return result of procedure invocation if any.

So, the role to provide a way to communicate with an application, which usually consists of multiple libraries.
Technically, it should not be difficult to provide to a user/client different interfaces to communicate with an application.
It should be quite similar, if an application exposes REST or SOAP, Protobuf interface or anything similar.
Corresponding handler just accepts input data, prepares required ones for a procedure, and call a procedure.

```
    | Controllers                          | Procedures               |
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
    | Other controllers                    | Other procedures         |
    +--------------------------------------+--------------------------+               



     REST     |-------+
                      |
     Thrift   |-----+ |
                    | |
     Protobuf |-----+-+------<->-| application's procedure
                    | |
     ....     |-----+ |
                      |
     RMI      |-------+    

```

## How much logic should be in a controller??
To answer that question I would consider few aspects:
- Single responsibility principle from OOP design
- Testability

Probably even the second aspect is already redundant if consider just only "Single responsibility principle".
The principle is described as following at [oodesign.com](https://www.oodesign.com/single-responsibility-principle)

>> A responsibility is considered to be one reason to change. This principle states that if we have 2 reasons to change for a class, 
>> we have to split the functionality in two classes.

It means if mix up in one controller validiting input, validating authentication, preparing other parameters for a procedure call,
converting output, then such implementation clearly violates of a "Single responsibility principle". 
It is much better to move out logic, which do not belong to a controller, from a controller to abstractions, input processing chains.

From prospective of testing a controller that has long and complicated implementation with many dependencies is a nightmare :)
Using Test-driven development and keeping in mind Single responsibility principle helps to create small enough implementation that are easy to test and
comply to OOP design patterns.

Very good example of such architecture is Spring Boot - validation can be done by providing special annotation on DTO fields, filters handle an authentication
and assign all the parameters required by a context to a request attributes which can be easily used in a controller. Validation rules are subject on unit test
A controller becomes easy to test, because all what we need to test is how a controller accepts input data, fails on validation, and if output complies 
expected format. The rest of the things are tested separately by unit tests and finally by integration tests.
