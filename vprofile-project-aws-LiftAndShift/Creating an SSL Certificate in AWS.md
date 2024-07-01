# Linking Your Domain to a Hosting Service & SSL Certificate Creation

## Linking AWS Route53 to your Domain Hosting Service

1. Log into your [AWS account](https://signin.aws.amazon.com/)
2. Locate **Route53 > Get started > Create hosted zones** and add your domain name and hit the **Create hosted zone** button
3. Then go to your domain hosting service, mine is [GoDaddy](https://www.godaddy.com/)
4. Log in and navigate to **Domain(s)** > ***your-domain*** > **Manage DNS** > **Nameservers**
5. Hit the **Change Nameservers** button
6. Go to AWS console copy listed nameservers under **Value/Route traffic to** field
7. In GoDaddy copy all the nameservers from AWS one by one and hit **Save** button
8. On the pop up menu (**Name server Update)** check the box and hit the **Continue** button

## Create SSL Certificate
1. Login in your [AWS account](https://signin.aws.amazon.com/)
2. Search for **AWS Certificate Manager** and make sure you are on the same region where you deployed your resource.
3. Hit the **Request a certificate** button
4. Click on **Request a public certificate** and hit **Next**
5. Enter your domain name under **Domain names**
6. Leave the default selection under **Validation method**
7. Leave the default selection under **Key algorithm**
8. Click on the **Request**

**Note**: It takes 10 min to populate.
