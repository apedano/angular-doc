# 04A-Signals as data state handler

https://medium.com/ngconf/keeping-state-with-a-service-using-signals-bee652158ecf

In this article, I will tackle creating a service to store state with signals since this is one of the most common uses of the `BehaviorSubject` as you can see in some of the articles below:

[Angular State Management With BehaviorSubject](https://dev.to/ngconf/angular-state-management-with-behaviorsubject-22b0?source=post_page-----bee652158ecf--------------------------------)

[Angular service to handle state using BehaviorSubject](https://medium.com/ngconf/keeping-state-with-a-service-using-signals-bee652158ecf)

## Creating the service

We will start by defining the class that other services will extend in order to define the state they want to store. 
This class uses the `signalstate`, and it is initialized with an empty object of type `T`.

