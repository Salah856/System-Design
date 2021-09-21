# DESIGN A WEB CRAWLER

A web crawler is known as a robot or spider. It is widely used by search engines to discover new or updated content on the web. Content can be a web page, an image, a video, a PDF file, etc. A web crawler starts by collecting a few web pages and then follows links on those pages to collect new content. 

![image](https://user-images.githubusercontent.com/23625821/133965293-0d59d9e2-399d-4da2-aff3-67096ffe6136.png)


## Gneral features of a web crawler: 

- Scalability: The web is very large. There are billions of web pages out there. Web crawling should be extremely efficient using parallelization.
- Robustness: The web is full of traps. Bad HTML, unresponsive servers, crashes, malicious links, etc. are all common. The crawler must handle all those edge cases.

- Politeness: The crawler should not make too many requests to a website within a short time interval.
- Extensibility: The system is flexible so that minimal changes are needed to support new content types. For example, if we want to crawl image files in the future, we should not need to redesign the entire system.


## Workflow of a web crawler 

![image](https://user-images.githubusercontent.com/23625821/133965738-e1e85023-2646-49b0-9cf5-44335c4948d1.png)

Most important building components and techniques in depth:

- Depth-first search (DFS) vs Breadth-first search (BFS)
- URL frontier
- HTML Downloader

- Robustness
- Extensibility
- Detect and avoid problematic content


BFS is commonly used by web crawlers and is implemented by a first-in-first-out (FIFO) queue. In a FIFO queue, URLs are dequeued in the order they are enqueued. However, this implementation has two problems:

- Most links from the same web page are linked back to the same host. All the links in wikipedia.com are internal links, making the crawler busy processing URLs
from the same host (wikipedia.com). When the crawler tries to download web pages in parallel, Wikipedia servers will be flooded with requests. This is considered as “impolite”.

![image](https://user-images.githubusercontent.com/23625821/134113148-8a54f111-5154-41ea-8643-b74781e74e6e.png)

- Standard BFS does not take the priority of a URL into consideration. The web is large and not every page has the same level of quality and importance. Therefore, we may want to prioritize URLs according to their page ranks, web traffic, update frequency, etc.























