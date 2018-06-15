---
title: "An Interface For PageObjects and UI Controls"
date: 2018-06-15
categories: automation
---
In my work I have had to deal with automated tests that switch back and forth from interacting from Windows desktop programs and web applications in the browser. When the tests were using desktop programs, the used the .Net UI Automator framework to access UI controls. When the tests were useing web applications, they accessed controls using the Selenium WebDriver. This made me realise that PageObject classes should not depend on the way that UI controls are accessed. I wanted a PageObject class that could encapsulate both dekstop application and web application UIs. To do this, I created two interfaces, IPage<IUIElement> and IUIElement<T>.



```c#
public interface IPage<IUIElement>
{
	void RegisterID(string label, PageElementID id);	
	UIElement GetControl(string label);
}

public interface IUIElement<T>
{
	T GetElement();
	string Text();
	void Click();
	void Input(string text);
}

```



### UI Controls

IUIElement<T> provides methods to be implemented by classes that encapsulate the functionality of interacting with UIControls. I can create a class that implements this interface for  desktop applications like this. Since I use the .Net UI Automation framework, I give T the concrete type of System.Windows.Automation.AutomationElement.



##### IUIElement<System.Windows.Automation.AutomationElement>

```c#
using System.Windows.Automation;

public class DesktopElement : IUIElement<AutomationElement>
{
	public AutomationElement element;
	public DesktopElement(AutomationElement _element)
	{
		this.element = _element;
	}
	public AutomationElement GetElement()
	{
		return this.element;
	}

	public void Input(string text)
	{
		ValuePattern pattern = element.GetCurrentPattern(ValuePattern.Pattern) as ValuePattern;
		pattern.SetValue(text);
	}

	public string Text()
	{
		return null as string;
     }	

	public void Click()
	{
		InvokePattern pattern = element.GetCurrentPattern(InvokePattern.Pattern)						as InvokePattern;
		pattern.Invoke();
	}

	public void Select()
	{
		SelectionItemPattern pattern = element.GetCurrentPattern(SelectionItemPattern.Pattern) 				as SelectionItemPattern;
		pattern.Select();
	}
```



##### IUIElement<OpenQA.Selenium.IWebElement>

In the case of browser applications, we will assign to T a concrete value of OpenQA.Selenium.IWebElement, since we are using the Selenium WebDriver.



```c#
using OpenQA.Selenium.IWebElement;

public class WebElement : IUIElement<IWebElement>
{
	public IWebElement element;

	public WebElement(IWebElement _element)
	{
		this.element = _element;
	}
	
	public IWebElement GetElement()
	{
	return this.element;
	}

	public string Text()
	{
		return this.element.Text;
	}
	
	public void Click()
    {
    	this.element.Click();
    }

    public void Input(string text)
    {
        this.element.Click();
        this.element.Clear();
        this.element.SendKeys(text);
    }
}
```



Since WebElement and DesktopElement both implent IUIElement, my automated tests will not have to infer whether the UI control is in the desktop or browser to interact with it. It just needs to know that the object implements IUIElement<T>.



### PageObjects

I Our classes that represent UI pages will implement the methods of IPage<UIElement> access a UI Control. The advantage of using an interface is so that  my tests do not have to worry about whether the UI is a desktop application or in the web browser.



##### IPage.RegisterID() and PageElementID

The purpose of RegisterID() is to associate a ubiquitous string with a PageElementID object.   The PageElementID object contains two properties: Value and Type. This is so that when writing tests, I don't have to explicitly state what property should be used to identify a UI control. 



```c#
public class PageElementID
{
    public string Type;
    public string Value;

    public PageElementID(string _Value, string _Type)
    {
        this.Type = _Type;
        this.Value = _Value;
    }
}
```



The implementation of GetControl() should use both of these properties to discern the correct way to access a UI Control. 



##### DesktopPage: IUIElement<System.Windows.Automation.AutomationElement>

For example, a desktop UI control might have an AutomationID property defined, or it might have its Name property defined. The DesktopPage class' implementation of Get Control first gets the PageElementID, then will search for the UI Control by evaluating each control's Name property or AutomationID property, depending on the value that assigned to PageElementID.Type.

Here is my class that implements IPage for desktop UIs. 



```c#
public class DesktopPage : IPage<DesktopElement>
{
	public Dictionary<string, PageElementID> LabelToID;
	public UIMap UIMap;
	public DesktopPage(UIMap _UIMap)
	{
		this.LabelToID = new Dictionary<string, PageElementID>();
		this.UIMap = _UIMap;
	}
	
	public void RegisterID(string label, PageElementID id)
	{
		this.LabelToID.Add(label, id);
	}

	public DesktopElement GetControl(string label)
	{
		PageElementID id = this.LabelToID[label];
		if (id.Type.Equals("AutomationID"))
		{
			return new 		DesktopElement(this.UIMap.SearchForControlById(id.Value).Result);
		}
		else if (id.Type.Equals("Name"))
		{
			return new DesktopElement(this.UIMap.SearchForControlByName(id.Value).Result);
		}
		return null as DesktopElement;
	}
}
```



##### WebPage: IUIElement<System.Windows.Automation.AutomationElement>

in the case of web pages, GetControl() can be implemented such that it uses PageElementID to indicate whetther PageElementID.Value should be interpreted as a element id or a CssSelector. 



```c#
public class AwesomePage : IPage<WebElement>
{
	public IWebDriver driver;
	public Dictionary<string, PageElementID> LabelToId;
	public AwesomePage(IWebDriver _driver)
	{
		this.driver = _driver;
		this.LabelToId = new Dictionary<string, PageElementID>();
	}
	public void RegisterID(string label, PageElementID id)
	{
		this.LabelToId.Add(label, id);
	}

	public PageElementID GetID(string label)
	{
		try
		{
			return this.LabelToId[label];
		}
		catch (KeyNotFoundException)
		{
			throw new IDNotRegisteredException(label);
		}
	}

	public WebElement GetControl(string label)
	{
		PageElementID id = this.GetID(label);
		if (id.Type.Equals("id"))
		{
			return new WebElement(this.driver.FindElement(By.Id(id.Value)));
		}
		if (id.Type.Equals("CssSelector"))
        {
        	return new WebElement(this.driver.FindElement(By.CssSelector(id.Value)));
        }
		return null as WebElement;
	}
}
```

