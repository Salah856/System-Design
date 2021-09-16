# DESIGN A URL SHORTENER

Here are the basic use cases:

1. URL shortening: given a long URL => return a much shorter URL. 
2. URL redirecting: given a shorter URL => redirect to the original URL. 
3. High availability, scalability, and fault tolerance considerations. 


## URL redirecting
Once the server receives a tinyurl request, it changes the short URL to the long URL with 301 redirect.

![image](https://user-images.githubusercontent.com/23625821/133559109-27b61b17-0fcb-4c14-b01d-2b336ac254f3.png)

![image](https://user-images.githubusercontent.com/23625821/133559179-4d1df5bb-4f5c-4c8f-b88e-064025b8d0a3.png)


A 301 redirect shows that the requested URL is “permanently” moved to the long URL. Since it is permanently redirected, the browser caches the response, and subsequent requests for the same URL will not be sent to the URL shortening service. Instead, requests are redirected to the long URL server directly.


A 302 redirect means that the URL is “temporarily” moved to the long URL, meaning that subsequent requests for the same URL will be sent to the URL shortening service first. Then, they are redirected to the long URL server.

