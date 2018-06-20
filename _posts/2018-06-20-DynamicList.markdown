---
title: Creating a PageObject to handle dynamic lists
date: 2018-06-20
categories: automation
---

When writing automated tests, I often encounter a list or menu whose items change depending on the configuration and state of the application. I could keep track of the state of the application and create a PageObject for each possible menu, but that would get messy. Instead, I could use the selenium WebDriver to get the text labels of each menu item, and then create a structure that maps each label to its associated item.

For example, Let's say that we want to represent a list where each menu item element has a span tag and the class attribute "title". The Each menu item is also a child element of a parent div whose id attribute is "FileList". 


``` html
<div id="FileList">
    <span class=".title">File A</span>
    <span class=".title">File B</span>
    <span class=".title">File C</span>
    <span class=".title">File D</span>
    <span class=".title">File E</span>					
</div>
```


The CSS Selector for these list items will be:


```
#FileList > .title
```


I can write a class that  in C# that represents any list whose items can be identified by a CSS Selector. When the list is instantiated, it will look up all the IWebElements by using that selector, and map the element's text to the IWebElement. 


```c#
using OpenQA.Selenium;


public class DynamicList
{
	private IWebDriver WebDriver;
	private Dictionary<string, IWebElement> ListItems
	private string CSSSelector;
	
    public DynamicList(IWebDriver webDriver, string cssSelector)
    {
        this.WebDriver = webDriver;
        this.CssSelector = cssSelector;
        this.ListItems = new Dictionary<string, IWebElement>();
        this.PrepareListItems();
    }
    
    public PrepareListItems()
    {
        IList<IWebElement> elements = webDriver.FindElements(By.CssSelector(this.cssSelector));
        foreach (IWebElement element in elements)
        {
            this.ListItems.Add(element.Text, element);
        }
    }
    
    public IWebElement this[string listText]
    {
        get
        {
            return this.ListItems[listText];
        }
        set
        {
            this.ListItems.Add(element.Text, element);
        }
    }
}
```


The Dynamic List Class will be used for the file list example like this:


```c#
using OpenQA.Selenium.Chrome;

IWebDriver WebDriver = new ChromeDriver();
WebDriver.Navigate().GoToUrl("...");
DynamicList FileList = new DynamicList(ChromeDriver, "#FileList > .title");


FileList["File A"].Click();
```

