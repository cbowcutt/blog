---
layout: post
title: Using the Selenium WebDriver to drive existing Chrome processes
---
# Using the Selenium WebDriver to drive existing Chrome processes

When I write automated tests for web applications with the Selenium WebDriver, the browser might either be in a desktop Chrome session, or embedded in a desktop application using Cefsharp.

When the automated tests are for a desktop chrome session, setting up the webdriver instance is simple. All you need to do is specify the path of the chromedriver executable on your machine, and assuming that chrome is installed, the webdriver instance will open up a new chrome browser on its own. This is because the chromedriver assumes that the chrome executable is located (in Windows) under C:\Users%USERNAME%AppData\Local\Google\Chrome\Application\Chrome.exe. The default location on linux is /usr/bin/google-chrome.



```c#
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;

string chromeDriverPath = @"C:\chromedriver.exe";
IWebDriver driver = new ChromeDriver(chromeDriverPath);

```



In my work experience, I have had to run automated tests inside a CefSharp browser embedded inside a desktop applications. This means that I had to make it so that the WebDriver instance used an already running browser process instead of creating a new one. In order to do this, the ChromeDriver constructor can be overridden by passing in a ChromeOptions object as an argument. 

##### ChromeOptions

A ChromeDriver object can connect to an existing chrome process by setting the BinaryLocation and DebuggerAddress properties on the ChromeOptions object used by the ChromeDriver. BinaryLocation is a string specifies the location of the executable associated with the running browser process. DebuggerAddress is a string specifies the address of the Chrome debugger server to connect to.



```c#
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;

ChromeOptions customOptions = new ChromeOptions();
options.BinaryLocation = @"C:\CefSharp.BrowserSubprocess.exe";
options.DebuggerAddress = @"localhost:8181";

IWebDriver driver = new ChromeDriver(customOptions);
```

