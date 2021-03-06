---
title: "Using Windows UI Automation"
date: 2018-06-12
categories: automation
---
# Using the Windows UI Automation in automated tests for Windows desktop applications  

## Introduction

This tutorial shows how the Windows UI Automation framework can be used to programmatically retrieve UI components and perform actions in a desktop application.

## Overview of the framework

The Windows UI Automation framework allows developers writing automated tests to programmatically access Windows UI controls. It is included in the  Many windows desktop UI frameworks, like WPF, already provide an interface for UI Automation, so nothing special has to be done to the application under test to support it. 



## How to locate UI Controls in your tests

#### The UI Automation Tree and AutomationElement.RootElement

Each UI component in the windows desktop can be exposed to your tests as an instance of the AutomationElement class. All the AutomationElements on the Windows desktop are organized in a tree structure, and the root element can be accessed by calling AutomationElement.RootElement. 



#### AutomationElement.FindAll() and AutomationElement.FindFirst()

Every instance of AutomationElement has a FindAll() and FindFirst() method. These methods have two parameters that specify the scope of the search, and the criteria that the AutomationElements within the scope must meet in order to be returned. Both of these methods start their search at the instance of AutomationElement on which it was called. FindAll() returns a ReadOnlyCollection of AutomationElements, while FindFirst() returns the first AutomationElement encountered that meets the criteria.

All UI Controls on the desktop can be can be accessed by calling the FindAll() or FindFirst() on RootElement. These methods have two parameters.



#### System.Windows.Automation.TreeScope

The first parameter is a TreeScope object. This defines the scope where the method will search for AutomationElements. TreeScope.Children can be passed in as the first argument to FindAll() to search all child elements adjacent the the instance of AutomationElement on which the method is called. TreeScope.Descendants can be passed in as the first argument to FindAll() to search all child elements that are reachable from the instance of AutomationElement on which the method is called.


```c#
using System.Windows.Automation;
AutomationElement rootElement = AutomationElement.RootElement;

AutomationElementCollection immediateChildren = rootElement.FindAll(TreeScope.Children, Condition.True);
AutomationElementCollection children = rootElement.FindAll(TreeScope.Descendants,
Condition.True);
```



#### System.Windows.Automation.Condition


The second parameter is of the type System.Windows.Automation.Condition. This parameter specifies the criteria that a UI Control within the specified scope must meet in order to be returned.

The PropertyCondition class is a useful subclass of the Condition class that can be used to specify the value that a UI Control's property must have in order to returned. The constructor for instantiating a PropertyCondition objects takes in an AutomationProperty object and an object named Value. 

For example, if there is a UI Control whose Name property is "OK", we can create a PropertyCondition and then use it as the second argumeent in FindFirst() like this:

```c#
using SystemWindows.Automation;

Condition nameIsOKCondition = new PropertyCondition(AutomationElement.NameProperty, "OK" as object);
AutomationElement elementOK = AutomationElement.RootElement.FindFirst(TreeScope.Children, nameIsOKCondition);
```

This next example will assign to allInvokableControls a collection of all UI controls where the InvokePattern is available. InvokePattern is a subclass of ControlPattern that "Represents controls that initiate or perform a single, unambiguous action and do not maintain state when activated." These are mainly used to represent buttons.

```c#
using System.Windows.Automation;

PropertyCondition propertyCondition = new PropertyCondition(AutomationElement.IsInvokePatternEnabledPorperty, true);
AutomationElementCollection allInvokableControls = rootElement.FindAll(TreeScope.Children, propertyCondition);
```


#### Making the search for UI Controls much more efficient

The Automation Tree can contain thousands of elements, depending on how many desktop windows and applications are open. If the scope of the search gets too big, FindFirst() and FindAll() can cause a stack overflow. Fortunately, the Automation Tree provides three default views that can greatly filter the elements in the Automation Tree so that it only contains UI Controls. When it comes to UI automation, we are only concerned with these controls. These elements will have their IsControlElement property set to true. 

#### TreeViewWalker

The TreeViewWalker class provides methods to navigate the Automation Tree. It is not as efficient as using AutomationElement.FindAll() or AutomationElement.FindFirst(), the TreeViewWalker methods can result in creating cross-process calls, but it does provide ways to filter the Automation Tree so that only UI Controls are encountered. 

#### TreeViewWalker.ControlViewWalker

TreeViewWalker is a static class that contains field named ControlViewWalker. ControlViewWalker is a predefined TreeWalker that contains only control elements on a UI, starting from a particular element.


## Using ControlPattern to interact with UIControls

AutomationElements themselves do not directly provide functionality for performing user actions. Instead,  Control Patterns are used to implement this. Control Patterns are a collection of classes and interfaces that are used to expose a UI control's functionality, such as clicking or entering text. A Control Pattern for a UI control can be accessed through the instance of AutomationElement that represents that control. Each subclass of ControlPattern has its own methods for the types of actions that a user can perform on a UI Control

#### Accessing a ControlPattern object from an AutomationElement Instance

A ControlPattern can be accessed from an AutomationElement instance by calling GetCurrentPattern() on the instance. This method takes one parameter, an AutomationPattern object that identifies a control pattern.  An AutomationElement instance can support multiple types of patterns. GetSupportedPatterns() can return a collection of ControlPatterns that the instance will support.



```c#
AutomationPattern[] patternsSupportedByRootElement = AutomationElement.RootElement.GetSupportedPatterns();
```



#### Programmatically click a button using the InvokePattern class
Buttons commonly use the InvokePattern when clicking the button has a single, unambiguous action. Given an instance of AutomationElement that represents a button, buttonElement, the InvokePattern for that element can be retrieved from buttonElement. Invoke() can be called on the InvokePattern instance to activate the action that is performed when the button is clicked.



```c#
InvokePattern invokePattern = buttonElement.GetCurrentPattern(InvokePattern.Pattern) as InvokePattern;
invokePattern.Invoke(); // click the button
```



#### Programmattically insert text into a textbox using the ValuePattern class

The ValuePattern class can be used to programmatically input a string value into a UI Control that has a text input. The example below shows how the program can input the text "HELLO" into the UI Control represented by textInputElement. 

```c#
AutomationElement textInputElement = AutomationElement.RootElement.FindFirst(TreeScope.Descendants, new PropertyCondition(AutomationElement.IsValuePatternAvailableProperty, true));
ValuePattern pattern = textInputElement.GetCurrentPattern(ValuePattern.Pattern) as Pattern;
pattern.SetValue("HELLO");
```

