```
title: The Abstract Factory Design Pattern and Applying it to Page Objects
date: 06-27-2018
category: automation
```

Sometimes we want to create Page Objects for web applications or desktop applications. Both of these applications are essentially 
interacted with in the same way; Buttons are clicked on, text is read and entered, pictures are displayed, and so on. However, when
we automate the interaction of these applications, the implementation of how these interactions are executed differ. In the case of web 
applications, we may use the Selenium WebDriver or some other framework. For desktop applications, we may use the Microsoft UI Automation 
framework.

Since the functionality of these two different types of page objects are the same but the implementation is distinct, it makes sense to 
create an abstract Page Class and two subclasses for both web and desktop applications. When we use these PageObjects in our automated tests,
we will then not have to worry if the application we are working against is a a desktop or web application. However, the creation of these
page objects still will differ. This is where the Abstract Factory design pattern makes itself useful.

If we were to create an abstract PageObjectFactory class and two subclasses, HTMLPageFactory and DesktopPageFactory, then we could alleviate
our test code of needing knowledge beforehand whether the desired page it wants is for a web or desktop application. This will also make
construction of the pages uniform.


Here is what the abstract PageObjectFactory and the abstract PageObject class will look like.

```c#
public abstract class PageFactory
{
	public abstract Page MakePage();
	public abstract IInvokable CreateInvokable(IPageElementID PageElementID);
	public abstract ISelectable CreateSelectable(IPageElementID PageElementID);
	public abstract IReadable CreateReadable(IPageElementID PageElementID);
	public abstract IInputable CreateInputable(IPageElementID PageElementID);
	public abstract IExpandable CreateExpandable(IPageElementID PageElementID);
}
  
  
public abstract class Page
{
	public Dictionary<string, IInvokable> Invokables;
	public Dictionary<string, ISelectable> Selectables;
	public Dictionary<string, IReadable> Readables;
	public Dictionary<string, IInputable> Inputables;
	public Dictionary<string, IExpandable> Expandables;

	public abstract IInvokable Invokable(string stringID);
	public abstract ISelectable Selectable(string stringID);
	public abstract IReadable Readable(string stringID);
	public abstract IInputable Inputable(string stringID);
	public abstract IExpandable Expandable(string stringID);
}
```


The abstract Page class declares members that will map strings to a certain PageElement. It also provides methods for 
accessing a certain PageElement.


The PageFactory declares methods for creating a Page object and methods for creating different types of PageElements that can be used by 
other classes.


The PageFactory class is abstract, so it can't be used directly. Instead, subclasses, HTMLPageFactory and DesktopPageFactory are created
along with HTMLPage and desktopPage.


Here is the implementation of HTMLPage and HTMLPageFactory.


```c#
public class HTMLPage : Page
{
	public IWebDriver Driver;

	public HTMLPage(IWebDriver driver)
	{
		this.Driver = driver;
		this.Invokables = new Dictionary<string, IInvokable>();
		this.Selectables = new Dictionary<string, ISelectable>();
		this.Readables = new Dictionary<string, IReadable>();
		this.Inputables = new Dictionary<string, IInputable>();
		this.JavaScriptPageElements = new Dictionary<string, JavaScriptPageElement<object>>();
	}

	public override ISelectable Selectable(string stringID)
	{
		return this.Selectables[stringID];
	}

	public override IInvokable Invokable(string stringID)
	{
		return this.Invokables[stringID];
	}

	public override IInputable Inputable(string stringID)
	{
		return this.Inputables[stringID];
	}

	public override IReadable Readable(string stringID)
	{
		return this.Readables[stringID];
	}
	public override IExpandable Expandable(string stringID)
	{
		return this.Expandables[stringID];
	}

}
  
public class HTMLPageFactory : PageFactory
{
	private IWebDriver Driver;
	public HTMLPageFactory(IWebDriver driver)
	{
		this.Driver = driver;
	}
	public override Page MakePage()
	{
		return new HTMLPage(this.Driver);
	}
	private IWebElement GetElement(IPageElementID pageElementID)
	{
		return this.Driver.FindElement((By)pageElementID.Locator());
	}

	public override IInvokable CreateInvokable(IPageElementID pageElementID)
	{
		return new HTMLInvokable(this.GetElement(pageElementID));
	}

	public override ISelectable CreateSelectable(IPageElementID pageElementID)
	{
		return new HTMLRadioButton(this.Driver.FindElement((By)pageElementID.Locator()));
	}
	public override IReadable CreateReadable(IPageElementID pageElementID)
	{
		return new HTMLReadable(this.Driver.FindElement((By)pageElementID.Locator()));
	}

	public override IInputable CreateInputable(IPageElementID pageElementID)
	{
		return new HTMLInputable(this.Driver.FindElement((By)pageElementID.Locator()));
	}

	public override IExpandable CreateExpandable(IPageElementID pageElementID)
	{
		return new HTMLExpandable(this.GetElement(pageElementID));
	}

	public JavaScriptPageElement<T> CreateJavaScriptPageElement<T>(string expression)
	{
		return new JavaScriptPageElement<T>(Evaluate<T>(expression));
	}
	public T Evaluate<T>(string expression)
	{
		return OpenQA.Selenium.Support.Extensions.WebDriverExtensions.ExecuteJavaScript<T>(Driver, expression);
	}
}
```  


Here is the implementation for DesktopPageFactory and DesktopPage.


```c#
public class DesktopPageFactory : PageFactory
{
	private UIMap UIMap;
	public DesktopPageFactory(UIMap uiMap)
	{
		this.UIMap = uiMap;
	}
	public override Page MakePage()
	{
		return new DesktopPage(this.UIMap);
	}
	private AutomationElement GetElement(IPageElementID pageElementID)
	{
		return this.UIMap.GetG4Window().Result.FindFirst(TreeScope.Descendants, (Condition)pageElementID.Locator());
	}

	public override IInvokable CreateInvokable(IPageElementID pageElementID)
	{
		return new DesktopInvokable(this.GetElement(pageElementID));
	}

	public override ISelectable CreateSelectable(IPageElementID pageElementID)
	{
		return new DesktopSelectable(this.GetElement(pageElementID));
	}
	public override IReadable CreateReadable(IPageElementID pageElementID)
	{
		return new DesktopReadable(this.GetElement(pageElementID));
	}

	public override IInputable CreateInputable(IPageElementID pageElementID)
	{
		return new DesktopInputable(this.GetElement(pageElementID));
	}

	public override IExpandable CreateExpandable(IPageElementID pageElementID)
	{
		return new DesktopExpandable(this.GetElement(pageElementID));
	}

	public ISelectable CreateDesktopCheckbox(IPageElementID pageElementID)
	{
		return new DesktopCheckbox(this.GetElement(pageElementID));
	}
}

public class DesktopPage : Page
{
	public UIMap UIMap;
	public DesktopPage(UIMap uiMap)
	{
		this.UIMap = uiMap;
		this.Invokables = new Dictionary<string, IInvokable>();
		this.Selectables = new Dictionary<string, ISelectable>();
		this.Readables = new Dictionary<string, IReadable>();
		this.Inputables = new Dictionary<string, IInputable>();
		this.Expandables = new Dictionary<string, IExpandable>();
	}

	public override ISelectable Selectable(string stringID)
	{
		return this.Selectables[stringID];
	}

	public override IInvokable Invokable(string stringID)
	{
		return this.Invokables[stringID];
	}

	public override IReadable Readable(string stringID)
	{
		return this.Readables[stringID];
	}

	public override IInputable Inputable(string stringID)
	{
		return this.Inputables[stringID];
	}

	public override IExpandable Expandable(string stringID)
	{
		return this.Expandables[stringID];
	}
}

public class DesktopPageElementID : IPageElementID
{
	public Condition Value;

	public DesktopPageElementID(Condition _value)
	{
		this.Value = _value;
	}

	public Object Locator()
	{
		return this.Value;
	}
}
```

The two definitions for the Page subclasses are very similar.  The only difference is that DesktopPage has a member of type UIMap, 
while HTMLPage has a member of IWebDriver (This makes me wonder if I even need to subclass Page at all).


We can use the Abstract Page Factory to create a Page Object for a web applciation like this:


```c#
public Page OKButtonPage()
{
  IWebDriver driver = new IWebDriver();
  PageFactory factory = new HTMLPageFactory(driver);
  Page page = new factoryMakePage();
  page.Invokables.Add("OKButton", factory.CreateInvokable(new HTMLPageElement(By.Id("OK"));
  return page;
}

```


The code above represents the construction of a Page object that has an OK button. A test method that clicks the OKButton can use this 
method like this:


```c#
[TestMethod]
public Page Test_OKButton()
{
  Page okPage = OKButtonPage();
  okPage.Invokable("OKButton").Invoke();
}

```

Now, let's say that the developers change this page from HTML to a Windows desktop application. We then can change the method 
OKButtonPage() to handle this.

```c#
public Page OKButtonPage()
{
  IWebDriver driver = new IWebDriver();
  PageFactory factory = new DesktopPageFactory(driver);
  Page page = new factoryMakePage();
  PropertyCondition condition = new PropertyCondition(AutomationElement.NameProperty, "OK);
  page.Invokables.Add("OKButton", factory.CreateInvokable(new HTMLPageElement(condition);
  return page;
}
```


Since the modified version of OKButtonPage() still returns a page object, we do not have to change our test code at all. 


