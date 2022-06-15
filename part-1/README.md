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

For the case of a single server, the cheapest possible EC2 instance we can use is the t4g.nano instance ($0.0042 per hour => ~$3.00 a month). There are also data transfer costs out of AWS, so depending on the amount of traffic, you may need to account for the $0.09 per GB transferred out of EC2. This would essentially amount to the size of the files you vend out multiplied by the number of website visitors. For more info on EC2 pricing, see the [AWS EC2 On-Demand Pricing page](https://aws.amazon.com/ec2/pricing/on-demand/).
  
(instert diagram w/ load balancer)
  
For the case with the load balancer, our new cost is the cost of the load balancer plus the hourly cost of the EC2 instance type multiplied by the number of hosts. The data transfer cost is still the same, just split across hosts. The load balancer cost is $16.20 per month for hourly rate and an additional cost relative to the amount of traffic and size of data being passed. For smaller applications, users probably would not incur significant costs for # of connnections and amount of data transferred. You can see more about Load Balancer pricing on the [AWS ELB Pricing page](https://aws.amazon.com/elasticloadbalancing/pricing/).
  
### Simple S3 Static Site Model
  
This is the most basic S3 static site we can build. With this model, we don't have a custom domain name and we don't support an HTTPS connection to our site, however, we will address this with the future models. The HTML/JS/CSS files are simply stored in an S3 bucket and the bucket is public so the user can access the resources, specifically something like the "index.html" file, directly. For pricing, we don't have to worry about hosting, as that is covered by Amazon S3. Each time the user loads a file, it will count as a GET request to AWS S3. The cost for GET requests to S3 is $0.0004 per 1000 requests, or $1.00 for every 2.5 million requests. Your site can be optimized by reducing the number of files to be downloaded for each page load (minimizing number of images, CSS files, JavaScript files).

(insert diagram with S3 here)
  
### S3 With CloudFront and Custom Domain
  
Here is the more complete version of an S3 static website. Here we will be using 3 more AWS Services: 1) Cloudfront - which is a CDN (Content Distribution Network) but also acts as a proxy for adding TLS to enable HTTPS, 2) ACM - AWS Certificate Manager creates and provides the public TLS/SSL certificates to Cloudfront to enable TLS/SSL, 3) Route53 - DNS service where we can buy a domain, create DNS records to point to our Cloudfront endpoint, and use the domain name for the ACM certificate as well.
  
(insert full diagram here)

Along with the custom domain name and HTTPS, there are cost benefits to this as well. First, it stands to reason if you're making a site, you are likely to buy a domain name for any of the scenarios above, so we can assume domain name cost is fixed and a non-factor in choosing a solution. Second, ACM is free. Last, Cloudfront comes with a great free tier (see [Cloudfront Pricing](https://aws.amazon.com/cloudfront/pricing/)) and because the CDN will cache the files from S3, you will have reduced cost from the $1.00 for every 2.5 million requests above as well as the $0.09 per GB of data transfer.
