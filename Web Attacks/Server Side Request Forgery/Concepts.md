SSRF (Server-Side Request Forgery) abuses the functionality of a server to perform internal or external requests on behalf of the server. For example, it can access internal servers that are not accessible to the public because the requests are being made from an internal resource.

A SSRF attack can

- Interact with internal systems
- Perform port scanning to discover internal services
- Data Exposure
- Denial of Service (DoS)
- Remote Code Execution (RCE)

Suppose _Bob_ (the hacker) is browsing the victim shopping website at _https://insecureweb.com_. _Bob_ notices that the application uses a backend API to fetch and display product images. The API takes a URL as input to determine which image to display. The URL used in the API request to fetch product images looks like this:

- **productImage=http://images.product/product?productID=1**

However this input is not properly sanitized, and _Bob_ can perform an SSRF attack and access the admin panel by changing the URL to http://localhost/admin. The request will looks like this:

- **productImage=http://localhost/admin**

_Bob_ successfully accessed the admin panel because the input was not properly sanitized, and the server displayed the admin panel content, believing the request originated from a trusted source.

![[SSRC.png]]
