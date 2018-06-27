---
title: "Advantages of Using a Wrapper Class for PageElements"
date: 2018-06-15
categories: automation
---
In my work I have had to deal with automated tests that switch back and forth from interacting from Windows desktop programs and web applications in the browser. When the tests were using desktop programs, the used the .Net UI Automator framework to access UI controls. When the tests were useing web applications, they accessed controls using the Selenium WebDriver. This made me realise that PageObject classes should not depend on the way that UI controls are accessed. I wanted a PageObject class that could encapsulate both dekstop application and web application UIs. To do this, I created a wrapper class for UI controls, and an interface for each type of interaction to be performed on the UI controls.



```c#
public class PageElement<T>
{
	public T Element;
	public PageElement(T element)
	{
		this.Element = element;
	}
	public T GetElement()
	{
		return this.Element;
	}
}

	public interface IInvokable { void Invoke(); }

	public interface IReadable { string Read(); }

	public interface IInputable { void Input(string value); }

	public interface ISelectable {
		void Select();
		void Deselect();
	}
```


The generic type T for PageElement is meant to be the type of the native UI Control used in other frameworks. When I am working with web pages., I can assign to T the concrete type of IWebElement.


```c#
IWebElement rocko = WebDriver.FindElement(By.Id("rocko"));
PageElement<IWebElement> rockoElement = new PageElement<IWebElement>(rocko);
```


In the case of wrapping Windows desktop applications, I can assign to T the concrete type of AutomationElement.


```c#
PropertyCondition locator = new PropertyCondition(AutomationElement.NameCondition, "rocko"));
AutomationElement rocko = AutomationElement.RootElement.FindFirst(TreeScope.Descendants, locator);
PageElement<AutomationElement> rockoElement = new PageElement<AutomationElement>(rocko);
```


I can create subclasses of the PageElement class that also implements each of the user UI Control interfaces. the code example below shows the class definitions for readable page elements and clickable (invokable) page elements. 

```c#
	public class HTMLReadable : PageElement<IWebElement>, IReadable
	{
		public HTMLReadable(IWebElement webElement) : base(webElement)
		{
			this.Element = webElement;
		}
		public string Read()
		{
			return this.Element.Text;
		}
	}
	
	public class HTMLInvokable : PageElement<IWebElement>, IInvokable
	{
		public HTMLInvokable(IWebElement webElement) : base(webElement)
		{
		}
		public void Invoke()
		{
			this.Element.Click();
		}
	}
	
	public class DesktopInvokable : PageElement<AutomationElement>, IInvokable
	{
		public DesktopInvokable(AutomationElement element) : base(element)
		{
		}

		public void Invoke()
		{
			(this.Element.GetCurrentPattern(InvokePattern.Pattern) as InvokePattern).Invoke();
		}
	}
	
	public class DesktopReadable : PageElement<AutomationElement>, IReadable
	{
		public DesktopReadable(AutomationElement element) : base(element)
		{
		}

		public string Read()
		{
			return (this.Element.GetCurrentPattern(TextPattern.Pattern) as TextPattern).ToString();
		}
	}
```

I stay away from the "one-size-fits-all" approach, because not all UI Control actions will not be valid on each control. Instead, I isolate the interfaces. At the same time, even though each class implements a unique interface, they can all be considered the same type since they are all subclasses of PageElement. 

This provides an advantage, because when I create instances of these classes in my automated tests, my test code only has to know whether it a UI control that can be read or clicked, and it doesn't have to worry if it is from a Windows application or a web application. 

This also makes my test code more maintanable. If the developers decide to replace the desktop UI with a full web UI, I won't have to change my test code that references those UI Controls. Instead, I will only have to modify the PageElement that encapsulates it.
