---
layout: post
title: How Google's PageRank Algorithm Actually works?
subtitle: Simplified
categories: Web
tags: [explained, computer-science, google, seo]
---

When you perform a basic google search, how does google determine in what order to show you your results?

_Google did try to explain us [here](https://www.youtube.com/watch?v=0eKVizvYSUQ)_

One of the most important algorithms Google uses to do this is called **PageRank** and it's an algorithm that attempts to estimate the importance of a website.

What does it mean for a website to be important ?

Well, the web consists of pages and those **pages can be connected to one another** via links and the PageRank algorithm generally assumes that if many other pages are linking to a particular page then that page is probably important. So a web page that is linked to more would be considered more important than a webpage that is linked to fewer pages.

![img1](https://github.com/AnimeshRy/blog/blob/master/assets/images/article5/img1.png?raw=true)

**BUT** there's a problem with that approach the problem is that it's easy for someone to **artificially inflate** their own web page's importance. For Example, If I wanted to make my webpage seem more important in the eyes of PageRank I could just create lots of other pages that all linked to my website using that strategy I could make my website seem as important as I wanted it to be.

So to really define what it means for a page to be important we need to mold our definition a little bit more. A page is more important the more it is linked to by other important pages but this definition **seems a bit circular,** how can we calculate a page's importance if doing so requires knowing the importance of other pages?

Well one way to calculate this is using what's known as the [Random Surfer Model](https://en.wikipedia.org/wiki/Random_surfing_model).

The idea is this - Imagine someone browsing the web, they begin on some page chosen at random and then they randomly pick a link from that page to another page that they visit. The _Random Surfer_ keeps repeating this process.

Pick a link randomly → visit a newpage → pick another link → visit another page.

The idea now is to keep score i.e maintain a count of how many times our random surfer visits each page. Each time they land on a new page we will update that page's score. Pages that have more links to them are more likely to be visited so they'll eventually have higher scores and because those pages are more likely to be visited the pages they link to are also more likely to be visited. So we linked from a more important page will matter more than a link from a less important page.

After we continue this process for a while we can take a look at the resulting scores and calculate what percent of the total score each page have, this gives us some measure for the relative importance of these pages represented as what percent of the time a random surfer on the Internet can be expected to be on that page.

![gif1](https://github.com/AnimeshRy/blog/blob/master/assets/images/article5/Page_1.gif?raw=true)

There is still one problem with this approach though and it's the fact that pages on the Internet might not all be connected to each other.

Imagine a network of pages like the one below, if we randomly start on this page and we keep following links. We will only ever visit one set of pages on the web completely ignoring the rest of the internet.

Since none of the other pages are reachable via any of the links from the pages that were currently visited.

![img2](https://github.com/AnimeshRy/blog/blob/master/assets/images/article5/img2.png?raw=true)

To solve this problem we need to occasionally reset our _Random Web Surfing_ we do this by introducing what's called a **[Damping Factor](https://en.wikipedia.org/wiki/Damping_factor).**

Suppose the damping factor is 0.85, which means that 85% of the time our _random surfer_ will follow a link from the page that is currently off as they were doing. 15 percent of the time though our random surfer will instead switch to a page on the internet **chosen completely** at random. With enough time this ensures that we will eventually explore all parts of this network i.e _The Internet_ of webpages and not get stuck at one particular set.

This model lets us know to take any network of webpages and calculate the relative importance of those pages in the first few steps the random surfer takes, the numbers aren't particularly accurate a lot is based just on random chance but with enough time the random surfer will continue to explore more and more and the numbers will eventually converge to a stable PageRank value for each page and those values can then be used to determine what order search results should appear in with the more important pages appearing the first.

PageRank isn't the only way to calculate the importance of web pages but it's a pretty effective way and it ensures that for the most part when you search for something the results you get are hopefully the results you actually want.

Author - [Animesh Singh](https://iamanimesh.tech)
