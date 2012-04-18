#Udacity-Challenge
#=================

#Here is my attempt to win the challenge with an autocompletion feature for my search engine.

def compute_ranks(graph):
    d = 0.8 # damping factor
    numloops = 10
    
    ranks = {}
    npages = len(graph)
    for page in graph:
        ranks[page] = 1.0 / npages
    
    for i in range(0, numloops):
        newranks = {}
        for page in graph:
            newrank = (1 - d) / npages
            
            sum = 0
            
            for node in graph:
                if page in graph[node]:
                    outlinks = len(graph[node])
                    sum = sum + (d * ranks[node]) / outlinks
            
            newrank += sum
            
            newranks[page] = newrank
        ranks = newranks
    return ranks

def crawl_web(seed): # returns index, graph of inlinks
    tocrawl = [seed]
    crawled = []
    graph = {}  # <url>, [list of pages it links to]
    index = {} 
    while tocrawl: 
        page = tocrawl.pop()
        if page not in crawled:
            content = get_page(page)
            add_page_to_index(index, page, content)
            outlinks = get_all_links(content)
            graph[page] = outlinks
            union(tocrawl, outlinks)
            crawled.append(page)
    return index, graph


def get_next_target(page):
    start_link = page.find('<a href=')
    if start_link == -1: 
        return None, 0
    start_quote = page.find('"', start_link)
    end_quote = page.find('"', start_quote + 1)
    url = page[start_quote + 1:end_quote]
    return url, end_quote

def get_all_links(page):
    links = []
    while True:
        url, endpos = get_next_target(page)
        if url:
            links.append(url)
            page = page[endpos:]
        else:
            break
    return links


def union(a, b):
    for e in b:
        if e not in a:
            a.append(e)

def add_page_to_index(index, url, content):
    words = content.split()
    for word in words:
        add_to_index(index, word, url)
        
def add_to_index(index, keyword, url):
    if keyword in index:
        index[keyword].append(url)
    else:
        index[keyword] = [url]

def lookup(index, keyword):
    if keyword in index:
        return index[keyword]
    else:
        return None
    
def get_page(url):
    if url in cache:
        return cache[url]
    else:
        print "Page not in cache: " + url
        return None
    
cache = {
   'http://udacity.com/cs101x/urank/index.html': """<html>
<body>
<h1>Dave's Cooking Algorithms</h1>
<p>
Here are my favorite recipies:
<ul>
<li> <a href="http://udacity.com/cs101x/urank/hummus.html">Hummus Recipe</a>
<li> <a href="http://udacity.com/cs101x/urank/arsenic.html">World's Best Hummus</a>
<li> <a href="http://udacity.com/cs101x/urank/kathleen.html">Kathleen's Hummus Recipe</a>
</ul>

testIndex

For more expert opinions, check out the 
<a href="http://udacity.com/cs101x/urank/nickel.html">Nickel Chef</a> 
and <a href="http://udacity.com/cs101x/urank/zinc.html">Zinc Chef</a>.
</body>
</html>






""", 
   'http://udacity.com/cs101x/urank/zinc.html': """<html>
<body>
<h1>The Zinc Chef</h1>
<p>
I learned everything I know from 
<a href="http://udacity.com/cs101x/urank/nickel.html">the Nickel Chef</a>.
</p>
<p>
For great hummus, try 
<a href="http://udacity.com/cs101x/urank/arsenic.html">this recipe</a>.

testZinc

</body>
</html>






""", 
   'http://udacity.com/cs101x/urank/nickel.html': """<html>
<body>
<h1>The Nickel Chef</h1>
<p>
This is the
<a href="http://udacity.com/cs101x/urank/kathleen.html">
best Hummus recipe!
</a>
testNickel
</body>
</html>






""", 
   'http://udacity.com/cs101x/urank/kathleen.html': """<html>
<body>
<h1>
Kathleen's Hummus Recipe
</h1>
<p>
testKathleen
<ol>
<li> Open a can of garbonzo beans.
<li> Crush them in a blender.
<li> Add 3 tablesppons of tahini sauce.
<li> Squeeze in one lemon.
<li> Add salt, pepper, and buttercream frosting to taste.
</ol>

</body>
</html>

""", 
   'http://udacity.com/cs101x/urank/arsenic.html': """<html>
<body>
<h1>
The Arsenic Chef's World Famous Hummus Recipe
</h1>
<p>
testArsenic
<ol>
<li> Kidnap the <a href="http://udacity.com/cs101x/urank/nickel.html">Nickel Chef</a>.
<li> Force her to make hummus for you.
</ol>

</body>
</html>

""", 
   'http://udacity.com/cs101x/urank/hummus.html': """<html>
<body>
<h1>
Hummus Recipe
</h1>
<p>
testHummus
<ol>
<li> Go to the store and buy a container of hummus.
<li> Open it.
</ol>

</body>
</html>




""" ,
}

    
    
def startWith(key,word):
    length = len(word)
    if key >= word:
        if key[0:length] == word:
            return True    
        
def quicksort(data):   # [[k,u,r],[k,u,r],...]
    if len(data) == 1:
        return data[0]
    if len(data) == 0:
        return []
    pivot = data[0]  #[k,u,r]
    worse = []
    better = []
    
    for elem in data[1:]:
        if elem[2] <= pivot[2]:
            worse.append(elem)
        else:
            better.append(elem)  
    
    return quicksort(better) + data[0] + quicksort(worse)
    

def autocomplete(index, graph, ranks, word):
    if len(word) == 0:
        return None
    
    key_list = []
    for key in index:
        if startWith(key,word):
            key_list.append(key)
    
    #key_list now contains all possible keys that start with word
    
    data = []
    for elem in key_list:
        for url in index[elem]:
            data.append([elem,url,ranks[url]])
    
    #data contains [[keyword, url, rank],...]

    sorted_data = quicksort(data)
    
    #sorted_data contains a list in following structure [keyword,url,rank,keyword,url,rank...]
    #the highest rank is the first one  
    
    #return the best 3 keywords with the highest ranking pages
    best_keywords = []
    
    i=0
    k=0
    while i<3 and k<len(sorted_data):
        if sorted_data[k] not in best_keywords:
            best_keywords.append(sorted_data[k])
            k = k + 3 # jump to next keyword
            i = i + 1
        else:
            k = k+3
    
    
    return best_keywords
    



index, graph = crawl_web('http://udacity.com/cs101x/urank/index.html')
ranks = compute_ranks(graph)

print autocomplete(index,graph,ranks, "test")
# I manipulated the cache websites, each of them has a testXXXXXX word in it, so when a user types "test" into the search enginge, he should get autocompletion
# for the most important website (highest rank), which is in this case kathleen.html 
# This will return:
# ['testKathleen', 'testNickel', 'testArsenic']
