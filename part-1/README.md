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
  
  
## Architecture

### Traditional Server Model

We will first consider the case using a traditional website hosted by a single server, then by multiple servers behind a load balancer. For cost analysis, we will be looking at AWS EC2 prices.
  
(insert diagram w/o load balancer)

For the case of a single server, the cheapest possible EC2 instance we can use is the t4g.nano instance ($0.0042 per hour => ~$3.00 a month). There are also data transfer costs out of AWS, so depending on the amount of traffic, you may need to account for the $0.09 per GB transferred out of EC2. This would essentially amount to the size of the files you vend out multiplied by the number of website visitors. For more info on EC2 pricing, see the [AWS EC2 On-Demand Pricing page](https://aws.amazon.com/ec2/pricing/on-demand/)
  
(instert diagram w/ load balancer)
  
For the case with the load balancer, our new cost is the cost of the load balancer plus the hourly cost of the EC2 instance type multiplied by the number of hosts. The data transfer cost is still the same, just split across hosts. The load balancer cost is $16.20 per month for hourly rate and an additional cost relative to the amount of traffic and size of data being passed. For smaller applications, users probably would not incur significant costs for # of connnections and amount of data transferred. You can see more about Load Balancer pricing on the [AWS ELB Pricing page](https://aws.amazon.com/elasticloadbalancing/pricing/)
