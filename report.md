A program that interacts with websites is called a __web crawler__. Web crawlers are used to create search engine indexes, to archive pages, and in our case: to explore Wikipedia. 

## Breaking the Problem into Steps
The steps our crawler program will execute. The manual process we followed was this:

1. Open an article
2. Find the first link in the article
3. Follow the link
4. Repeat this process until we reach the Philosophy article, or get stuck in an article cycle.

The key phrase in this process is "Repeat". This four-step process is essentially a loop! If our program duplicates the manual process, our program will consist of one big loop. To implement the crawling process we will use a while loop.

## Recording our Progress
As we loop, we'll need to keep track of the articles we visit so we can output the path our crawler finds. I'll add this into our process:

1. Open an article
2. Find the first link in the article
3. Follow the link
4. Record the link in the ```article_chain``` data structure.
5. Repeat this process until we reach the Philosophy article, or get stuck in an article cycle.


```article_chain``` will be our program's output. We can also use it to keep up with which article to explore next. In step 1 of the loop, out program can open the article that's at the end of the article chain.  

Our algorithm needs to keep track of the chain of articles that have already been seen, so that they can be outputted at the end. It also needs to keep track of the last article seen, so that it can be used as input for the next iteration of the while loop.
A ```list``` can store the article chain in the correct order, and it's easy to select the last article from the list.


Circumstances under which we should end the loop:

- we reach Philosophy,
- we reach a page we've already visited, hence find ourselves in a cycle of articles,
- we go on for too long (we 25 steps is plenty, but can be adjusted), or
- we find a page that has no links on it - we simply can't keep going in this case.

## The Sequence of Steps
Establishing the steps that need to go in the loop:

1. Download the HTML for the current article.
2. Find the first link in the current article's HTML.
3. Add the first link from the current article to article_chain

There's one other step:

4. Pause for a couple seconds so we don't flood Wikipedia with requests.

When we manually found links and clicked them, our speed was naturally limited by how fast we could read and click. Our Python program won't be constrained this way, it will loop as quickly as pages can be downloaded. While this is a time saver, it's impolite to hammer a web server with rapid repeated requests. If we don't slow our loop down the server might think we're attackers trying to overload the server, and block us. And the server might be right! If there's a bug in our code we might get into an infinite loop and drown the server with requests. To avoid this we should include a pause in the main loop. (Websites specify their automated crawler policies in a __robots.txt__ file.

[Wikipedia's robots.txt](https://en.wikipedia.org/robots.txt) : Wikipedia says that, "Friendly, low-speed bots are welcome viewing article pages…". By including a delay, we comply with Wikipedia's terms!).


Describe the sequence of steps using the following pseudo code.
```
page = a random starting page  
article_chain = [ ]  
while title of page isn't 'Philosophy' and we have not discovered a cycle:  
    append page to article_chain  
    download the page content  
    find the first link in the content  
    page = that link  
    pause for a second  

```

### The continue_crawl Function

```continue_crawl(search_history, target_url)```

- ```search_history``` is a list of strings which are the urls of Wikipedia articles. The last item in the list most recently found url.  

- ```target_url``` is a string, the url of the article that the search should stop at if it is found.

```continue_crawl```should return __True__ or __False__ following these rules:

- if the most recent article in the search_history is the target article the search should stop and the function should return __False__.
- If the list is more than 25 urls long, the function should return __False__.
- If the list has a cycle in it, the function should return __False__.
- otherwise the search should continue and the function should return __True__.

```{py}
def continue_crawl(search_history, target_url, max_steps = 25):
    if search_history[-1] == target_url:
        print("We've found the target article.")
        return(False)
    elif len(search_history)>max_steps:
        print("The list is more than 25 urls long. The search has gone on suspiciously long; aborting search!")
        return(False)
    elif search_history[-1] in search_history[0:-1]:
        print("We've arrived at an article we've already seen; aborting search!.")
        return(False)
    else:
        return(True)

````