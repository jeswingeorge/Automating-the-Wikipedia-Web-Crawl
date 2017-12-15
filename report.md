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

```

Here are those four steps as comments in a skeleton of the while loop.

```
while continue_crawl(article_chain, target_url): 
    # download html of last article in article_chain
    # find the first link in that html
    # add the first link to article_chain
    # delay for about two seconds
```

The first step, __download html of last article in article_chain__, is where we'll be using the requests package to obtain the html from wikipedia. The second step, __find the first link in that html__, will involve parsing that html with BeautifulSoup to grab the URL of the first link.  
Putting these two steps together into a single function whose __input will be a string containing the URL for a wikipedia article__ and whose __output will be a string containing the URL of the first link in the body of that wikipedia article__. Let's call this function ```find_first_link```.

Now to add the first link to article chain and to give delay for about two seconds, the following code was added :

```{py}
import time

def web_crawl():
    while continue_crawl(article_chain, target_url): 
        # download html of last article in article_chain
        # find the first link in that html
        first_link = find_first_link(article_chain[-1])
        # add the first link to article chain
        article_chain.append(first_link)
        # delay for about two seconds
        time.sleep(2)
```

### Finding the First Link: First Attempt
The code used was :

```{py}
def  find_first_link(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    first_link = soup.find(id='mw-content-text').find(class_="mw-parser-output").p.a.get('href')
    return(first_link)
```

The [A.J.W. McNeilly wikipedia article](https://en.wikipedia.org/wiki/A.J.W._McNeilly) was pretty simple by Wikipedia standards. Articles with infoboxes, pronunciation guides, and inconveniently placed footnotes introduce new problems for us to solve.

### Finding the First Link: Second Attempt

The ```children``` method finds direct descendants. We can also use the ```find_all``` method and set the __recursive argument to False__ so that it only finds children.

Rewriting the find_first_link()

```{py}
def  find_first_link(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    content_div = soup.find(id='mw-content-text').find(class_='mw-parser-output')
    for element in content_div.find_all('p', recursive=False):
        if element.a:
            first_relative_link = element.a.get('href')
            break
            
    return(first_link)
```

The first line finds the div that contains the article's body. The next line loops over each ```<p>``` tag in the div, if that tag is a child of the div. The [documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#recursive) tells us that, "If you only want Beautiful Soup to consider direct children, you can pass in recursive=False."

The body of the loop checks to see if an __a__ tag is in the paragraph. If so, it gets the url from the link, stores it in __first_relative_link__, and ends the loop.

__Note__  : This could also have written using the children method. The body of the loop would be different.

### Finding the First Link: Third Attempt

```{py}
def  find_first_link(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    content_div = soup.find(id='mw-content-text').find(class_='mw-parser-output')
    for element in content_div.find_all('p', recursive=False):
        if element.find('a', recursive=False):
            first_relative_link = element.a.get('href')
            break

    return(first_relative_link)
```

This works because "special links" like footnotes and pronunciation keys all seem to be wrapped in more div tags. Since these special links aren't direct descendants of a paragraph tag, I can skip them using the same technique as before. I used the __find__ method this time rather than __find_all__ because find returns the first tag it finds rather than a list of matching tags.











