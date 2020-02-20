---
title: "Python Web Scraping 101"
date: 2019-08-21T16:13:28+02:00
highlight: true
image: "/img/profile_pic_400x400.jpg"
description: "In this post we will cover almost all the tools Python offers you to web scrape. We will go from the more basic to the most advanced one and will cover the pros and cons of each."
canonicalUrl: "https://www.scrapingninja.co/blog/web-scraping-101-with-python"
---



# Web Scraping 101 with Python

In this post we will cover almost all the tools Python offers you to web scrape. We will go from the more basic to the most advanced one and will cover the pros and cons of each. Of course, we won't be able to cover all aspect of every tool we discuss, but this post should be enough to have a good idea of which tools does what, and when to use which.

*Note: when I talk about Python in this blog post you should assume that I talk about Python3.* 

## Web Fundamentals

The internet is **really complex**: there are many underlying technologies and concepts involved to view a simple web page in your browser. I don’t have the pretension to explain everything, but I will show you the most important things you have to understand in order to extract data from the web.

### HyperText Transfer Protocol

HTTP uses a **client/server** model, where an HTTP client (A browser, your Python program, curl, Requests...) opens a connection and sends a message (“I want to see that page : /product”)to an HTTP server (Nginx, Apache...). 

Then the server answers with a response (The HTML code for example) and closes the connection. HTTP is called a stateless protocol, because each transaction (request/response) is independent. FTP for example, is stateful.

Basically, when you type a website address in your browser, the HTTP request looks like this:

<script src="https://gist.github.com/ScrapingNinjaHQ/b183f0cba458646eef4383ddc037d407.js"></script>

In the first line of this request, you can see multiples things:

- the GET verb or method being used, meaning we request data from the specific path: `/product/`.There are other HTTP verbs, you can see the full list [here](https://www.w3schools.com/tags/ref_httpmethods.asp).
- The version of the HTTP protocol, in this tutorial we will focus on HTTP 1.
- Multiple headers fields

Here are the most important header fields :

- **Host:** The domain name of the server, if no port number is given, is assumed to be 80**.**
- **User-Agent:** Contains information about the client originating the request, including the OS information. In this case, it is my web-browser (Chrome), on OSX. This header is important because it is either used for statistics (How many users visit my website on Mobile vs Desktop) or to prevent any violations by bots. Because these headers are sent by the clients, it can be modified (it is called “Header Spoofing”), and that is exactly what we will do with our scrapers, to make our scrapers look like a normal web browser.
- **Accept:** The content types that are acceptable as a response. There are lots of different content types and sub-types: **text/plain, text/html, image/jpeg, application/json** ...
- **Cookie** : name1=value1;name2=value2... This header field contains a list of name-value pairs. It is called session cookies, these are used to store data. Cookies are what websites use to authenticate users, and/or store data in your browser. For example, when you fill a login form, the server will check if the credentials you entered are correct, if so, it will redirect you and inject a session cookie in your browser. Your browser will then send this cookie with every subsequent request to that server.
- **Referrer**: The Referrer header contains the URL from which the actual URL has been requested. This header is important because websites use this header to change their behavior based on where the user came from. For example, lots of news websites have a paying subscription and let you view only 10% of a post, but if the user came from a news aggregator like Reddit, they let you view the full content. They use the referrer to check this. Sometimes we will have to spoof this header to get to the content we want to extract.

And the list goes on...you can find the full header list [here](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields).

A server will respond with something like this: 

<script src="https://gist.github.com/ScrapingNinjaHQ/63373f9ccc1215f2c9acdd106976f276.js"></script>

On the first line, we have a new piece of information, the HTTP code `200 OK`. It means the request has succeeded. As for the request headers, there are lots of HTTP codes, split into four common classes, 2XX for successful requests, 3XX for redirects, 4XX for bad requests (the most famous being 404 Not found), and 5XX for server errors.

Then, in case you are sending this HTTP request with your web browser, the browser will parse the HTML code, fetch all the eventual assets (Javascript files, CSS files, images...) and it will render the result into the main window.

In the next parts we will see the different ways to perform HTTP requests with Python and extract the data we want from the responses. 

### 1) Manually opening a socket and sending the HTTP request

### Socket

The most basic way to perform an HTTP request in Python is to open a [socket](https://docs.python.org/3/howto/sockets.html) and manually send the HTTP request.

<script src="https://gist.github.com/ScrapingNinjaHQ/f075ba18f4069be0e4c5ecbb2dcb16c6.js"></script>

Now that we have the HTTP response, the most basic way to extract data from it is to use regular expressions. 

### Regular Expressions

A regular expression (RE, or Regex) is a search pattern for strings. With regex, you can search for a particular character/word inside a bigger body of text.

For example, you could identify all phone numbers inside a web page. You can also replace items, for example, you could replace all uppercase tag in a poorly formatted HTML by lowercase ones. You can also validate some inputs ...

The pattern used by the regex is applied from left to right. Each source character is only used once. You may be wondering why it is important to know about regular expressions when doing web scraping?

After all, there is all kind of different Python module to parse HTML, with XPath, CSS selectors. 

In an ideal [semantic world,](https://en.wikipedia.org/wiki/Semantic_Web) data is easily machine-readable, the information is embedded inside relevant HTML element, with meaningful attributes.

But the real world is messy, you will often find huge amounts of text inside a p element. When you want to extract a specific data inside this huge text, for example, a price, a date, a name... you will have to use regular expressions.

**Note:** Here is a great website to test your regex: [https://regex101.com/](https://regex101.com/) and [one awesome blog](https://www.rexegg.com/) to learn more about them, this post will only cover a small fraction of what you can do with regexp.

Regular expressions can be useful when you have this kind of data:

    <p>Price : 19.99$</p>

We could select this text node with an Xpath expression, and then use this kind of regex to extract the price :

    ^Price\s:\s(\d+\.\d{2})\$

To extract the text inside an HTML tag, it is annoying to use a regex, but doable:

<script src="https://gist.github.com/ScrapingNinjaHQ/882ba6f43b81ba2204cb87f681fe3f02.js"></script>

As you can see, manually sending the HTTP request with a socket, and parsing the response with regular expression can be done, but it's complicated and there are higher-level API that can make this task easier. 

### 2) Using urllib3 & LXML

**Disclaimer**: It is easy to get lost in the urllib universe in Python. You have urllib and urllib2 that are parts of the standard lib. You can also find urllib3. urllib2 was split in multiple modules in Python 3, and urllib3 should not be a part of the standard lib anytime soon. This whole confusing thing will be the subject of a blog post by itself. In this part, I've made the choice to only talk about urllib3 as it is used widely in the Python world, by Pip and requests to name only them. 

Urllib3 is a high-level package that allows you to do pretty much whatever you want with an HTTP request. It allows doing what we did above with socket with way fewer lines of code.

<script src="https://gist.github.com/ScrapingNinjaHQ/1043855e1894caf699303a3820c29f12.js"></script>

Much more concise than the socket version. Not only that, but the API is straightforward and you can do many things easily, like adding HTTP headers, using a proxy, POSTing forms ... 

For example, had we decide to set some headers and to use a proxy, we would only have to do this.

<script src="https://gist.github.com/ScrapingNinjaHQ/e2ba1e4667afe2c8c3916c37f89e3c8f.js"></script>

See? Exactly the same number of line, however, there are some things that urllib3 does not handle very easily, for example, if we want to add a cookie, we have to manually create the corresponding headers and add it to the request. 

There are also things that urllib3 can do that requsts can't, creation and management of pool and proxy pool, control of retry strategy for example.

To put in simply, urllib3 is between requests and socket in terms of abstraction, although way closer to requests than socket.

This time, to parse the response, we are going to use the lxml package and XPath expressions.

### XPath

Xpath is a technology that uses path expressions to select nodes or node- sets in an XML document (or HTML document). As with the Document Object Model, Xpath is a W3C standard since 1999. Even if Xpath is not a programming language in itself, it allows you to write expression that can access directly to a specific node, or a specific node-set, without having to go through the entire HTML tree (or XML tree).

Think of XPath as regexp, but specifically for XML/HMTL.

To extract data from an HTML document with XPath we need 3 things:

- an HTML document
- some XPath expressions
- an XPath engine that will run those expressions

To begin we will use the HTML that we got thanks to urllib3, we just want to extract all the links from the Google homepage so we will use one simple XPath expression: `[//a](//a)` and we will use LXML to run it. LXML is a fast and easy to use XML and HTML processing library that supports XPATH. 

*Installation*:

    pip install lxml

Below is the code that comes just after the previous snippet:

<script src="https://gist.github.com/ScrapingNinjaHQ/99bf1395535578154daeeeab0e3f7eec.js"></script>

And the output should look like this:

<script src="https://gist.github.com/ScrapingNinjaHQ/3f8858b261e4063017109cc8fbfb3e1c.js"></script>

You have to keep in mind that this example is really really simple and doesn't really show you how powerful XPath can be (note: this XPath expression should have been changed to `[//a/@href](//a/@href)` to avoid having to iterate on `links` to get their `href` ).

If you want to learn more about XPath you can read [this good introduction](https://librarycarpentry.org/lc-webscraping/02-xpath/index.html). The LXML documentation is also [well written and is a good starting point](https://lxml.de/tutorial.html). 

XPath expresions, like regexp, are really powerful and one of the fastest way to extract information from HTML, and like regexp, XPath can quickly become messy, hard to read and hard to maintain.

### 3) Using request & Beautifulsoup

![](https://raw.githubusercontent.com/requests/requests/master/docs/_static/requests-logo-small.png)

[Requests](https://github.com/psf/requests) is the king of python packages, with more than 11 000 000 downloads, it is the most widly used package for Python. 

Installation: 

    pip install requests

Making a request with Requests (no comment) is really easy: 

<script src="https://gist.github.com/ScrapingNinjaHQ/37e3bd9e806bda585e88fe28649f5562.js"></script>

With Requests it is easy to perform POST requests, handle cookies, query parameters... 

**Authentication to Hacker News**

Let's say we want to create a tool to automatically submit our blog post to Hacker news or any other forums, like Buffer. We would need to authenticate to those websites before posting our link. That's what we are going to do with Requests and BeautifulSoup!

Here is the Hacker News login form and the associated DOM:

![](https://ksah.in/content/images/2016/02/screenshot_hn_login_form.png)

There are three `<input>` tags on this form, the first one has a type hidden with a name "goto" and the two others are the username and password.

If you submit the form inside your Chrome browser, you will see that there is a lot going on: a redirect and a cookie is being set. This cookie will be sent by Chrome on each subsequent request in order for the server to know that you are authenticated. 

Doing this with Requests is easy, it will handle redirects automatically for us, and handling cookies can be done with the *Session* object. 

The next thing we will need is BeautifulSoup, which is a Python library that will help us parse the HTML returned by the server, to find out if we are logged in or not.

Installation: 

    pip install beautifulsoup4

So all we have to do is to POST these three inputs with our credentials to the /login endpoint and check for the presence of an element that is only displayed once logged in:

<script src="https://gist.github.com/ScrapingNinjaHQ/1f70dc2186f7b3116e811033c473f6eb.js"></script>

In order to learn more about BeautifulSoup we could try to extract every links on the homepage. 

*By the way, Hacker News offers a [powerful API](https://github.com/HackerNews/API), so we're doing this as an example, but you should use the API instead of scraping it!* 

The first thing we need to do is to inspect the Hacker News's home page to understand the structure and the different CSS classes that we will have to select:

![](/static/img/post2/hacker_news_screenshot-475f78bf-c737-4a60-8c24-d0cc220d7219.jpg)

We can see that all posts are inside a `<tr class="athing">` so the first thing we will need to do is to select all these tags. This can be easily done with: 

    links = soup.findAll('tr', class_='athing')

Then for each link, we will extract its id, title, url and rank:

<script src="https://gist.github.com/ScrapingNinjaHQ/80f68bbea369cffb69d559a0a2371d5a.js"></script>

As you saw, Requests and BeautifulSoup are great libraries to extract data and automate different things by posting forms. If you want to do large-scale web scraping projects, you could still use Requests, but you would need to handle lots of things yourself. 

When you need to scrape a lots of webpages, there are many things you have to take care of:

- finding a way of parallelizing your code to make it faster
- handling error
- storing result
- filtering result
- throttling your request so you don't over load the server

Fortunately for us, tools exist that can handle those things for us.

### 4) Using Scrapy

![](https://secure.meetupstatic.com/photos/event/1/b/6/6/600_468367014.jpeg)

Scrapy is a powerful Python web scraping framework. It provides many features to download web pages asynchronously, process and save it. It handles multithreading, crawling (the process of going from links to links to find every URLs in a website), sitemap crawling and many more. 

Scrapy has also an interactive mode called the Scrapy Shell. With Scrapy Shell you can test your scraping code really quickly, like XPath expression or CSS selectors. 

The downside of Scrapy is that the learning curve is steep, there is a lot to learn. 

To follow up on our example about Hacker news, we are going to write a Scrapy Spider that scrapes the first 15 pages of results, and saves everything in a CSV file. 

You can easily install Scrapy with pip: 

    pip install Scrapy

Then you can use the scrapy cli to generate the boilerplate code for our project: 

    scrapy startproject hacker_news_scraper

Inside `hacker_news_scraper/spider` we will create a new python file with our Spider's code:

<script src="https://gist.github.com/ScrapingNinjaHQ/43764a914632ec165bad4bc8c57341e4.js"></script>

There is a lot of convention in Scrapy, here we define an Array of starting urls. The attribute name will be used to call our Spider with the Scrapy command line. 

The parse method will be called on each URL in the `start_urls` array

We then need to tune Scrapy a little bit in order for our Spider to behave nicely against the target website. 

    # Enable and configure the AutoThrottle extension (disabled by default)
    # See https://doc.scrapy.org/en/latest/topics/autothrottle.html
    AUTOTHROTTLE_ENABLED = True
    # The initial download delay
    AUTOTHROTTLE_START_DELAY = 5

You should always turn this on, it will make sure the target website is not slow down by your spiders by analyzing the response time and adapting the numbers of concurrent threads. 

You can run this code with the Scrapy CLI and with different output format (CSV, JSON, XML...):

    scrapy crawl hacker-news -o links.json

And that's it! You will now have all your links in a nicely formatted JSON file. 

### 5) Using Selenium & Chrome —headless

Scrapy is really nice for large-scale web scraping tasks, but it is not enough if you need to scrape Single Page Application written with Javascript frameworks because It won't be able to render the Javascript code. 

![](/static/img/post2/SinglePageDiagram-9ae99e86-e997-4e18-9da9-d7abba599b9b.png)

It can be challenging to scrape these SPAs because there are often lots of AJAX calls and websockets connections involved. If performance is an issue, you should always try to reproduce the Javascript code, meaning manually inspecting all the network calls with your browser inspector, and replicating the AJAX calls containing the interesting data.

In some cases, there are just too many asynchronous HTTP calls involved to get the data you want and it can be easier to just render the page in a headless browser. 

Another great use case would be to take a screenshot of a page, and this is what we are going to do with the Hacker News homepage (again !)

You can install the selenium package with pip: 

    pip install selenium

You will also need [Chromedriver](http://chromedriver.chromium.org/):

    brew install chromedriver

Then we just have to import the Webdriver from selenium package, configure Chrome with headless=True and set a window size (otherwise it is really small):

{{< gist ScrapingNinjaHQ 420e9a344deb531fec240ff22997918b >}}


You should get a nice screenshot of the homepage:

![](/static/img/post2/hn_homepage-bd6bd60d-8778-404b-a82c-39ba76728e14.png)

You can do many more with the Selenium API and Chrome, like :

- Executing Javascript
- Filling forms
- Clicking on Elements
- Extracting elements with CSS selectors / XPath expressions

Selenium and Chrome in headless mode is really the ultimate combination to scrape anything you want. You can automate anything that you could do with your regular Chrome browser. 

The big drawback is that Chrome needs lots of memory / CPU power. With some fine-tuning you can reduce the memory footprint to 300-400mb per Chrome instance, but you still need 1 CPU core per instance. 

If you want to run several Chrome instances concurrently, you will need powerful servers (the cost goes up quickly) and constant monitoring of resources. 

## Conclusion:

Here is a quick recap table of every technology we discuss about in this about. Do not hesitate to tell us in the comment if you know some ressources that you feel have their places here.

<script src="https://gist.github.com/ScrapingNinjaHQ/78eafa5228ebbc55daf36a64fca20c34.js"></script>

I hope you enjoyed this blog post, it was a quick introduction to the most used Python tools for web scraping. In the next posts we're going to go deeper on each individual tools or topics like XPath, CSS selectors. 

Happy Scraping

---

Originally published on [scrapinginja.co](https://www.scrapingninja.co/blog/web-scraping-101-with-python)