# Hosting a Static Website on AWS S3
In this tutorial, you will learn how to build a static website and host it on AWS S3. In addition, you will learn how to add a custom domain name to your site using AWS Route53 and also make your site secure (add TLS encryption) using AWS Cloudfront

#### Technologies used
- AWS: S3, Route53, and Cloudfront
- Code: HTML/CSS/JS

##### Table of Contents  
- [Intro](#intro)  
  - [Common Static Website Scenarios](#common)
- [Architecture](#architecture)
  - [Traditional](#traditional)
  - [Simple S3 Model](#simple)
  - [Full S3 Model](#full)
  - [Cost Comparison](#cost)
- [Tutorial](#tutorial)
  - [Creating an S3 Bucket](#step1)
  - [Making the S3 Bucket into a Static Site](#step2)
  - [Create a certificate with ACM (optional - Domain Name Required)](#step3)
  - [Create a Cloudfront CDN mapping to the S3 Bucket (optional - Domain Name Required)](#step4)
  - [Create a Route 53 DNS record mapping your Domain to Cloudfront Distribution (optional - Domain Name Required)](#step5)
  - [Disable Public Access to S3](#step6)

<a name="intro" />

## Intro
What technologies would you traditionally use to build a website? With Javascript there's Node and Express, in Python theres Django and Flask, and in Java there's Spring, and etcetera, the list goes on for each language. For each of these web frameworks, the user makes a request to the service and the response from the service is the HTML and whatever other files are necessary to render the web app.

![basic-server](https://user-images.githubusercontent.com/23093022/174227996-d12e4e9f-a2e5-4343-949c-970773e4a313.PNG)

Sometimes, these web apps are rendered dynamically, meaning that the request the user provides to the server will affect the response (HTML, CSS, etc.). 

![dynamic-response](https://user-images.githubusercontent.com/23093022/174228297-e8379962-7616-4c69-8804-88f97059b2d7.PNG)

Often times, these responses are static and don't depend on the user. We will be focusing on these use cases. <add diagram here>
In fact, many websites are static but the view rendered by the browser can be updated without refreshing the page. These are static websites that have dynamic updates, usually powered by REST API's. For most of the web frameworks listed above, the servers are usually expected to return both the static content as well as provide a REST interface so the static site can have dynamic elements. This will important for the 2nd part of the tutorial.

![api-call](https://user-images.githubusercontent.com/23093022/174228603-936b098b-3950-4249-989b-50ec3ae172ad.PNG)

<a name="common" />
  
### Common Use Cases of Static Websites
- Company landing pages
- Personal Portfolio Site
- Any site can be written as a static site! (with supporting API, of course)
  
<a name="architecture" />
  
## Architecture

<a name="traditional" />
  
### Traditional Server Model

We will first consider the case using a traditional website hosted by a single server, then by multiple servers behind a load balancer. For cost analysis, we will be looking at AWS EC2 prices.
  
![image](https://user-images.githubusercontent.com/23093022/174228064-f3b079a3-57ef-4b56-9a4f-3e27b86eaa8f.png)

For the case of a single server, the cheapest possible EC2 instance we can use is the t4g.nano instance ($0.0042 per hour => ~$3.00 a month). There are also data transfer costs out of AWS, so depending on the amount of traffic, you may need to account for the $0.09 per GB transferred out of EC2. This would essentially amount to the size of the files you vend out multiplied by the number of website visitors. For more info on EC2 pricing, see the [AWS EC2 On-Demand Pricing page](https://aws.amazon.com/ec2/pricing/on-demand/).
  
![load-balancer](https://user-images.githubusercontent.com/23093022/174228805-0168610a-e07d-40ea-9b60-6bebd6841408.PNG)

For the case with the load balancer, our new cost is the cost of the load balancer plus the hourly cost of the EC2 instance type multiplied by the number of hosts. The data transfer cost is still the same, just split across hosts. The load balancer cost is $16.20 per month for hourly rate and an additional cost relative to the amount of traffic and size of data being passed. For smaller applications, users probably would not incur significant costs for # of connnections and amount of data transferred. You can see more about Load Balancer pricing on the [AWS ELB Pricing page](https://aws.amazon.com/elasticloadbalancing/pricing/).

<a name="simple" />
  
### Simple S3 Static Site Model
  
This is the most basic S3 static site we can build. With this model, we don't have a custom domain name and we don't support an HTTPS connection to our site, however, we will address this with the future models. The HTML/JS/CSS files are simply stored in an S3 bucket and the bucket is public so the user can access the resources, specifically something like the "index.html" file, directly. For pricing, we don't have to worry about hosting, as that is covered by Amazon S3. Each time the user loads a file, it will count as a GET request to AWS S3. The cost for GET requests to S3 is $0.0004 per 1000 requests, or $1.00 for every 2.5 million requests. Your site can be optimized by reducing the number of files to be downloaded for each page load (minimizing number of images, CSS files, JavaScript files).

![S3-site](https://user-images.githubusercontent.com/23093022/174228952-e54afd61-53e0-4baa-914c-99433d2ccd1f.PNG)

 
<a name="full" />
  
### S3 With CloudFront and Custom Domain
  
Here is the more complete version of an S3 static website. Here we will be using 3 more AWS Services: 1) Cloudfront - which is a CDN (Content Distribution Network) but also acts as a proxy for adding TLS to enable HTTPS, 2) ACM - AWS Certificate Manager creates and provides the public TLS/SSL certificates to Cloudfront to enable TLS/SSL, 3) Route53 - DNS service where we can buy a domain, create DNS records to point to our Cloudfront endpoint, and use the domain name for the ACM certificate as well.
  
![full-diagram-static](https://user-images.githubusercontent.com/23093022/174229278-969b5349-83ce-494f-b585-ca2fc7456ea0.PNG)

Along with the custom domain name and HTTPS, there are cost benefits to this as well. First, it stands to reason if you're making a site, you are likely to buy a domain name for any of the scenarios above, so we can assume domain name cost is fixed and a non-factor in choosing a solution. Second, ACM is free. Last, Cloudfront comes with a great free tier (see [Cloudfront Pricing](https://aws.amazon.com/cloudfront/pricing/)) and because the CDN will cache the files from S3, you will have reduced cost from the $1.00 for every 2.5 million requests above as well as the $0.09 per GB of data transfer.
 
<a name="cost" />
  
### Cost Comparison

A proper cost comparison can't be done without knowing the number of files being vended per page load, the number of requests the traditional server can handle per minute, and the rate of requests actually coming into the site. However, there is likely a tipping point where choosing the traditional server approach is more effective.
  
The traditional server model is suited well for consistent, predictable high-load scenarios where the serverless model is suited better for low and inconsistent traffic scenarios.
  
<a name="tutorial" />
  
## Tutorial
  
For this tutorial, you will need to make your own AWS account. To complete the entire tutorial, you will be required to purchase a domain name, however completing the tutorial is optional.
 
<a name="step1" />
  
### 1 Creating an S3 Bucket
  1. Navigate to the AWS S3 Console
  1. Press "Create Bucket"
     - General Configuration: choose whatever name and region you would like.
     - "Block Public Access settings for this bucket": uncheck all the boxes.
     - Go ahead and finish creating the bucket, no need to change any additional settings now.
  1. Navigate to your bucket by clicking the link.
  ![bucket](https://user-images.githubusercontent.com/23093022/174226155-9e439a78-9635-4943-9946-00394d17c344.PNG)
  1. Add the HTML/CSS files from this repository to the bucket (`index.html`, `other.html`, and `other.css`).
  
<a name="step2" />
  
### 2 Making the S3 Bucket into a static site
  1. Navigate to your bucket, and go to the "Properties" tab. Scroll to the bottom towards static website settings.
     - Enable static web settings and change the root and error document to `index.html`. This will make the default page `index.html`, and when the user tries to navigate to a non-existent page, it will take them to `index.html`.
  ![enable-static-website](https://user-images.githubusercontent.com/23093022/174226196-d52282a2-fe0e-48dc-bd36-c0299ba9c826.PNG)

  1. Go to the "Permissions" Tab.
     - Edit the bucket policy and change it to the following (except replace with your bucket name):
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::static-site-tutorial/*"
        }
    ]
}
  ```
  1. Go back to the Properties tab, get the static website domain, and enjoy your S3 Static Site!!
  
<a name="step3" />
  
### 3 Create a certificate with ACM (optional - Domain Name Required)
  1. Navigate to the ACM Console and change your region in the upper right corner to `us-east-1`. This is important as CloudFront will only consume ACM certificates in NA with this region
  ![ACM-us-east-1](https://user-images.githubusercontent.com/23093022/174226128-67e67c13-5d96-4712-9cf5-53e5425f4b53.PNG)
  1. Request a certificate, make it public, and use DNS verification.
     - If you have a domain name (example: laboe.org), then you can put that as the fully qualified domain name field. You can also use a sub domain, such as `static.john.laboe.org`. 
  1. Navigate to the certificate you created to check it's status:
     - Under the "Domains" section, press "Create Records in Route53." This will automatically create new CNAME records which ACM can use to verify that you own the domain.
  
<a name="step4" />
  
### 4 Create a Cloudfront CDN mapping to the S3 Bucket (optional - Domain Name Required)
  1. Navigate to the Cloudfront console
  1. Create a new distribution:
     - In the first section
        - Choose the origin domain corresponding to your S3 bucket.
        - Leave the origin path blank.
        - You may change the name or leave it the same.
        - Enable using OAI to access the S3 Bucket and create a new OAI. Also allow Cloudfront to update the Bucket Policy. This will effectively undo the permissions to access the S3 bucket directly and only allow users to access through Cloudfront.
        - Keep the rest of the settings the same.
  ![cloudfront-first-section](https://user-images.githubusercontent.com/23093022/174226050-f00b1be0-942f-46d3-881e-098e9c02a45f.PNG)
     - Under the "Default Cache Behavior" section.
        - Make sure HTTP is redirected to HTTPS.
     - Under the "Settings Section"
        - You may choose to change the price class. I usually choose to use only North America and Europe.
        - Add Alternate Domain Names: use the same name as you used for your ACM certificate.
        - Select the SSL certificate: it should be the one you created with ACM.
  ![cloudfront-settings](https://user-images.githubusercontent.com/23093022/174226084-6d817f0f-129e-4473-abd6-0f98e35b21e5.PNG)

     - Create the Distribution.
  1. Go to the "Error Pages" Tab
     - "Create Custom Error Response"
     - For the HTTP error code, choose 404.
     - Use a custom error response, and choose the page path to be `index.html`. This way, if the user tries to enter a bad page, they're redirected to index.html.
     - Select the HTTP response code to be 200. It will not render the page if the response code isn't success.
  ![cloudfront-error-response](https://user-images.githubusercontent.com/23093022/174225767-0aee5744-2cbc-47ae-8e95-18733c452861.PNG)
  
<a name="step5" />
  
### 5 Create a Route 53 DNS record mapping your Domain to Cloudfront Distribution (optional - Domain Name Required)
  
  1. Navigate to the Route53 Console and go to "Hosted Zones". Find the Hosted Zone for your domain name and navigate to it.
  1. Create a new Record. 
     - Set the subdomain to whatever you chose for your ACM (`static.john` for me).
     - Make it an alias.
     - Select the CloudFront endpoint.
  
![route53-records](https://user-images.githubusercontent.com/23093022/174226672-b1a1ecc9-dc48-4e80-a9e8-6ab918df5a48.PNG)
  
<a name="step6" />
  
### 6 Disable Public Access to S3
  We need to finish by only allowing CloudFront to access our S3 bucket.
   
  1. Navigate to the S3 Console. Navigate to your bucket.
  1. Go to the permissions tab:
     - Under "Block Public Access", re-enable (re-check) all the boxes.
     - Under "Bucket Policy", you should see two statements: (1) for public access and the other (2) for CloudFront OAI access. You should delete the first one.
  
Now we are finished. Type your Domain name into your web browser and check out your site. See mine at https://static.john.laboe.org 
