# Hosting a Static Website on AWS S3
In this tutorial, you will learn how to build a static website and host it on AWS S3. In addition, you will learn how to add a custom domain name to your site using AWS Route53 and also make your site secure (add TLS encryption) using AWS Cloudfront

#### Technologies used
- AWS: S3, Route53, and Cloudfront
- Code: HTML/CSS/JS

## Intro
What technologies would you traditionally use to build a website? With Javascript there's Node and Express, in Python theres Django and Flask, and in Java there's Spring, and etcetera, the list goes on for each language. For each of these web frameworks, the user makes a request to the service and the response from the service is the HTML and whatever other files are necessary to render the web app.

(add diagram here)
  
Sometimes, these web apps are rendered dynamically, meaning that the request the user provides to the server will affect the response (HTML, CSS, etc.). 

(add diagram here)

Often times, these responses are static and don't depend on the user. We will be focusing on these use cases. <add diagram here>
In fact, many websites are static but the view rendered by the browser can be updated without refreshing the page. These are static websites that have dynamic updates, usually powered by REST API's. For most of the web frameworks listed above, the servers are usually expected to return both the static content as well as provide a REST interface so the static site can have dynamic elements. This will important for the 2nd part of the tutorial.

(add diagram here)
  
  
### Common Use Cases of Static Websites
- Company landing pages
- Personal Portfolio Site
- Any site can be written as a static site! (with supporting API, of course)
