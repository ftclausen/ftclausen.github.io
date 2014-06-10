---
layout: post
title: Selenium Webdriver PageFactory Worked Example
date: 2013-09-24 10:09
categories: selenium
---

I found that, while adequate, the standard [selenium examples][selenium-examples] for PageFactory were not quite
enough to smoothly get me started. After [asking around][question1] I saw that, contrary to the
example, the most common practice seems to be to use the PageFactory inside the
[PageObject's][pageobject] constructor rather than from the client class. I will illustrate this
with code to test Blackboard Learn, my [employer's product][learn] starting with a simple login test.
It is my hope that through more examples people can help get started more easily.

So, first the **client code** that uses this PageFactory generated PageObject

{% highlight java linenos %}
public class LearnQA {
    public static void main(String[] args) {
        WebDriver driver = new FirefoxDriver();
        String url = "https://test.example.com/";
        
        LearnLoginPage loginPage = new LearnLoginPage(driver, url);
        // Creds set elsewhere
        LearnHomePage homePage = loginPage.loginAs(USERNAME, PASS);

    }
    
}
{% endhighlight %}

Basically, with a nice neat PageObject all we need to do is instantiate the page, "LearnLoginPage"
in this example, and then perform the requested test such as logging in as a particular user. This
hides the complexity of the page from the user of the class and we can change things internally
without impacting anyone.

As for the **actual LearnLoginPage**, this uses the convenience of the PageFactory to reduce boiler
plate code. First the listing -

{% highlight java linenos %}
public class LearnLoginPage {
    @FindBy(how = How.CLASS_NAME, using = "login-page")
    private WebElement loginPageClass;
    @FindBy(how = How.ID, using = "user_id")
    private WebElement usernameBox;
    @FindBy(how = How.ID, using = "password")
    private WebElement passwordBox;
    @FindBy(how = How.NAME, using = "login")
    private WebElement loginButton;
    
    private WebDriver driver;
    private String url;

    protected LearnLoginPage(WebDriver driver, String url) {
        this.driver = driver;
        this.url = url;
        driver.get(url);
        PageFactory.initElements(driver, this);
        
        By loginPageTag = By.tagName("body");
        if (! "login-page".equals(driver.findElement(loginPageTag).getAttribute("class"))) {
            throw new IllegalStateException("This is not the login page");
        }
    }
    
    public LearnLoginPage typeUsername(String username) {
        usernameBox.sendKeys(username);
        return this;
    }
    
    public LearnLoginPage typePassword(String password) {
        passwordBox.sendKeys(password);
        return this;
    }
    
    public LearnHomePage submitLogin() {
        loginButton.submit();
        return new LearnHomePage(driver);
    }
    
    public LearnHomePage loginAs(String username, String password) {
        typeUsername(username);
        typePassword(password);
        return submitLogin();
    }
    
}
{% endhighlight %}

Instead of the tedious WebElement finding we can now use the "FindBy" annotations to pre-populate
the various web elements we are going to use. Such as the username/password fields and the login
button. I believe this results in much more readible code.

Finally we have a couple of methods to 

- Type the username - typeUsername
- Type the password - typePassword
- Submit the login - submitLogin
- And, finally, a convenience method that combines all of the above into one operation called
  "loginAs".

[selenium-examples]: https://code.google.com/p/selenium/wiki/PageFactory
[question1]: http://stackoverflow.com/questions/18854752/selenium-pagefactory-check-if-on-correct-page
[pageobject]: https://code.google.com/p/selenium/wiki/PageObjects
[learn]: https://www.blackboard.com/platforms/learn/overview.aspx
[fredrepo]: https://github.com/
