---
title: Mapping Configurations to Application Behavior
date: 2018-07-07
category: automation
---


In my work, I write tests for a device called an Sound Level Meter. This device has the ability  measure sound properties in an environment over a period of time. One of the features of the device is that the user can control whether to start, stop or pause recording.

One of the challenges of testing this feature is that the Sound Level Meter has many different configurations which affects how the device's state changes based on input. For example, the user can configure the meter to automatically store a sound measurement after the meter has been running for a length of time set by the user. Or, the user can configure the meter to only store measurements when the user does it manually. 

This means that my automated tests have to keep track of how user input (or lack thereof) affect the device's state. If there were just a handful of configurations, it might suffice to hard-code the device's behavior for each setting. Unfortunately, Configurations are often parameterized and there are too many possible combinations to hard-code.

Instead of hard-coding the meter's expected behavior, it is easier to provide functionality to the automated tests so that the tests code can compute the behavior based on the given configurations. This is done so by modeling the behavior of the device as a state machine. 

Although there are different configurations, the meter will always be in one of four states: Running, Paused, Stopped and Stored. However, the configurations do affect the criteria for state transitions. The test code can use a generalized state machine where the transition symbols are variables whose definition will be determined by the configuration.

Here is a diagram of the generalized state machine below:


![Generalized State Machine]({{site.baseurl}}{{site.images.GeneralizedStateMachine}})

