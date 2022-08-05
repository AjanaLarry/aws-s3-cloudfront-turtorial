# Hosting a Vue.js Single Page Application (SPA) on AWS Simple Storage Service (S3)

![Host-Static-Website-on-AWS.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Host-Static-Website-on-AWS.png)

Recently, I was responsible for deploying and hosting a Vue.js frontend application on AWS. This project required HTTPS, a custom domain, a hosting service for the deployment files, and a content delivery network. As such, I decided to share the process as a step-by-step tutorial to help others who may need it.

### **Objectives**

In this tutorial, we will do the following:

- Set up a custom domain name
- Set up an S3 bucket to store and host our application file
- Set up an SSL Certificate, and redirect all HTTP traffic to HTTPS
- Create an IAM policy, a user, and access credentials
- Programmatically access AWS using ***aws-cli*** command and deploy files to S3
- Set up a CloudFront distribution and resolve it to our custom domain name
- Set up a bucket policy to restrict access to bucket content

### **Requirements**

We need an AWS account to be able to host and secure our single-page application in the cloud. To accomplish our objectives, we will use the following services:

- **AWS S3:** since our application is static, we can host the files in an S3 bucket. In addition to being highly scalable, reliable, and delivering high performance, S3 is also highly secure.
- **AWS Certificate Manager:** this allows us to generate a custom SSL certificate for our distribution and redirect all HTTP traffic to HTTPS.
- **CloudFront:** is a Content Delivery Network (CDN) that securely delivers content globally, offering low latency and high transfer speed. It provides distribution for our site at edge locations worldwide.
- **IAM:** is used to create temporary credentials that programmatically access and deploy our files to AWS S3.
- **Route 53:** this provides domain registration, DNS & traffic management services. We’ll set up Route 53 to point to our CloudFront distribution.

Some of the benefits of using the above choice of services are:

- They are serverless
- Cost efficient and employs a pay-as-you-go model
- Scalable and high performance
- Ease of use and maintenance
- No upfront cost

See below the architecture pattern for our hosted single-page application on AWS Cloud.

![Architecture Diagram 1.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Architecture_Diagram_1.png)

By the end of this tutorial, you will have successfully hosted your application on AWS, and it will be globally available. Alright, let’s get started.

### Step 1: **Build the Site for Production**

Depending on how you’ve created your site, I’ll leave it to you to build the production codebase. I used Vue Cli to code my web application, so I will use the ***npm run build*** command to compile my production site.

![Building SPA 1.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Building_SPA_1.png)

We have successfully built the web application, and this created a **dist** directory.

![Building SPA 2.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Building_SPA_2.png)

### Step 2: Set up a Domain in Route 53

- **Register a domain: myexample.com** on AWS
    1. Navigate to Route 53 dashboard. Click on **Register domain**
    2. Enter a **domain name.** Check for availability, add to **cart**, then click on **continue**
        
        ![Step 1b_Choose a Domain Name 2.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_1b_Choose_a_Domain_Name_2.png)
        
    3. Fill out the contact details for the domain registrant, then click **continue**
        
        ![Step 1c_Fill Contact Details.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_1c_Fill_Contact_Details.png)
        
    4. Accept terms and conditions to proceed. Click **complete order**
    5. Your domain will appear as a pending request. You will receive an email confirmation with a verification link from Amazon Registrar.
    
- **Create a Hosted Zone in AWS:**
    1. Navigate to Route 53 dashboard**.** Click on **create hosted zone** 
    2. Enter your **domain name** and leave the type as **Public hosted zone.** Click on **create hosted zone**
        
        ![Step 1f_Create Hosted Zone 2.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_1f_Create_Hosted_Zone_2.png)
        

- **Change registrar DNS:** this change allows Route 53 to handle DNS. Set the DNS servers to the values in the NS record in the Hosted Zone you created in Route 53.
    1. Navigate to your hosted zone. Copy the value of the **NS record** created with the hosted zone.
        
        ![Step 1g_Created Hosted Zone details.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_1g_Created_Hosted_Zone_details.png)
        
    2. Replace the DNS servers with the hosted zone **NS record values**
        
        ![Step 1h_Update Registra DNS 2.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_1h_Update_Registra_DNS_2.png)
        

### Step 3: Host the single-page application in S3

- **Create a bucket**
    1. Navigate to Amazon S3. Click on **Create bucket**
    2. Enter your unique bucket name (**my-example-bucket**)
        
        ![Step 2a_Create S3 bucket.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_2a_Create_S3_bucket.png)
        
    3. Disable **Access Control List, t**hen uncheck **Block Public Access settings** (this grants public read access to this bucket)**.**
        
        ![Step 2aa_Allow Public Access.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_2aa_Allow_Public_Access.png)
        
    4. Click **Create bucket.**
    
- **Turn on static website hosting**
    1. Navigate to the **properties tab**
    2. Scroll down, then click on **Edit static website hosting**
    3. **Enable** static website hosting, then set both the index and error document to **index.html** (this tells S3 where the root of the web application lies)
        
        ![Step 2ba_Enable Static Web Hosting.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_2ba_Enable_Static_Web_Hosting.png)
        

- **Upload Files**: copy in the built website files (including your index.html, favicon, JS, CSS, and any images).
    
    ![Step 2cb_object after bucket policy update.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_2cb_object_after_bucket_policy_update.png)
    

- Navigate to the properties tab to get the **URL endpoint** to your website.
- Open the URL in a new tab to see the hosted web application. It will output an error as we are yet to set an explicit bucket policy.
    
    ![Step 2bb_403 Error due to no bucket policy.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_2bb_403_Error_due_to_no_bucket_policy.png)
    
- **Configure the bucket policy:** in the permissions tab, set the bucket policy to allow public access to the bucket.
    1. Navigate to the **permissions tab,** then ****Click **Edit Bucket policy**
        
        ![Step 2c_Create a bucket policy.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_2c_Create_a_bucket_policy.png)
        
    2. Paste the JSON policy, replacing **BUCKET-NAME-GOES-HERE** with the name of your S3 bucket:
    
    ```json
    {
      "Version":"2012-10-17",
      "Statement":[{
    	"Sid":"PublicReadGetObject",
            "Effect":"Allow",
    	  "Principal": "*",
          "Action":["s3:GetObject"],
          "Resource":["arn:aws:s3:::BUCKET-NAME-GOES-HERE/*"
          ]
        }
      ]
    
    ```
    
    1. Click on **Save changes**.

Now, our bucket is all set to be served up.

### Step 4: Set up IAM User for Programmatic Access to the S3 Bucket

This process enables us to programmatically access AWS S3 from our local machine using the **aws-cli**. We will use the credentials to show that we are authorized to make changes to the S3 bucket. To accomplish this, we need to create an IAM policy and user.

- Navigate to the **IAM dashboard**, then select **policies**
- On the policy page, click on **Create policy**
    
    ![Step 3a_Create IAM Policy.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_3a_Create_IAM_Policy.png)
    

- Click on the **JSON tab**. Paste in the JSON code, then replace **BUCKET-NAME-GOES-HERE** with your bucket name
    
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "VisualEditor0",
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:ListBucket",
                    "s3:DeleteObject",
                    "s3:GetBucketPolicy"
                ],
                "Resource": [
                    "arn:aws:s3:::BUCKET-NAME-GOES-HERE",
                    "arn:aws:s3:::BUCKET-NAME-GOES-HERE/*"
                ]
            }
        ]
    }
    ```
    

This policy grants read and write access to our S3 bucket

- Click on **Review Policy**, then provide a name for your policy. Proceed to click on **Create Policy**.
    
    ![Step 3ab_Confirm IAM Policy.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_3ab_Confirm_IAM_Policy.png)
    

- Next, we’ll create an IAM user and attach this policy to its set of permissions. We will use this user’s credentials to access S3 using **aws-cli**.
- Navigate back to the **IAM dashboard**, and click on **users**. Then, click on **Add users**
    
    ![Step 3b_Create New User.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_3b_Create_New_User.png)
    
- Enter a name for your user and check the ***Access Type*** box for **“Programmatic access”.** Click on **Next: Permissions**
    
    ![Step 3ba_Add New User.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_3ba_Add_New_User.png)
    
- Add the policy that we created above. Click on the option to **“Attach existing policies directly”**
    
    ![Step 3bb_Set Permissions.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_3bb_Set_Permissions.png)
    
- Click on **Next: Tags**. We don’t need to do anything here. Click on **Next: Review**.
- Click on **Create user** to finish. This step will create the user’s credentials and output an ***Access key ID*** and a ***Secret access key*** (we will use these keys to access our S3 bucket).

### Step 5: Deploy to S3 Bucket from the CLI

We’ll set up a local AWS profile using ***aws configure*** command with the credentials generated from the above step:

```json
~/$ **aws configure --profile S3-VueJs-User1**
AWS Access Key ID [************]: PASTE ACCESS KEY ID FROM AWS
AWS Secret Access Key [********]: PASTE SECRET ACCESS KEY FROM AWS
Default region name [None]: **us-east-1**
Default output format [None]: None
```

Running ***aws configure*** sets up credentials in a file stored at **~/.aws/credentials**. After setting up the AWS profile, we can proceed to access our S3 bucket from the CLI.

- Run the below command to list all objects in the created bucket (replace **BUCKET-NAME-GOES-HERE** with your bucket name)

```json
~/$ aws --region us-east-1 --profile S3-VueJs-User1 s3 ls s3://BUCKET-NAME-GOES-HERE
```

- The above command should do nothing as our bucket is currently empty. It means we now have access to the S3 bucket (the IAM user and permissions were correct).
- Now proceed to deploy the production server (**./dist** ) to our S3 bucket using the sync command (replace **BUCKET-NAME-GOES-HERE** with your bucket name)

```json
~/$ aws --region us-east-1 --profile S3-VueJs-User1 s3 sync ./dist s3://BUCKET-NAME-GOES-HERE –delete
```

- The command above would upload each of the files in the **./dist** folder to the S3 bucket
    
    ![Step 4b_Deploy Dist to S3.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_4b_Deploy_Dist_to_S3.png)
    
- On the AWS console, navigate to the S3 and confirm the uploaded files are in the created bucket
    
    ![Step 4c_Deployed SPA.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_4c_Deployed_SPA.png)
    
- Next, click on the properties tab for the S3 bucket. Scroll down to static website hosting to locate the bucket website endpoint URL. Open the URL in your browser, and you will see that your application is live.
    
    ![Step 4c_Deployed SPA Page.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_4c_Deployed_SPA_Page.png)
    

**Note:** One of the limitations of static web hosting is that S3 website endpoints do not support HTTPS.

### Step 5: Request SSL Certificate using AWS Certificate Manager (ACM)

ACM offers free public certificates to provide Secure Socket Layer (SSL) and Transport Layer Security (TSL) encryption for your websites over the internet.

- Navigate to **ACM,** then click **Request a certificate**
    
    ![Step 5b_ Request SSL Certificate.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_5b__Request_SSL_Certificate.png)
    
- Set the domain name to your registered domain name(**myexample.com**)
- Click **DNS validation** and click on **Request**
    
    ![Step 5c_ Request Public Certificate.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_5c__Request_Public_Certificate.png)
    
- Click on **Create a record in Route53**.
    
    ![Step 5d_ Validate Certificate.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_5d__Validate_Certificate.png)
    
- This creates a CNAME record in the DNS settings for your domain, according to the values given to you here, and verifies that your own the domain. Check ACM to confirm the status of the issued certificate.
    
    ![Step 5f_Isssued SSL Certificate.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_5f_Isssued_SSL_Certificate.png)
    

### Step 6: Set up our CloudFront Distribution

CloudFront is a Content Delivery Network (CDN) that caches and serves copies of our websites at edge locations, thereby making our website faster to access. CloudFront also enables us to configure HTTPS while improving the performance of our site.

- **Create the First Distribution**
    1. Navigate to **Amazon CloudFront**, then click on **Create a CloudFront distribution**
    2. Configure the **Origin Domain Name** to your S3 bucket URL endpoint (**my-example-bucket.s3.us-east-1.amazonaws.com**)
    3. Click **Create new** **origin access identity (OAI):** CloudFront will use the OAI to access the content of our S3 bucket
    4. Click **Yes, update the bucket policy**
        
        ![Step 6cd_Updated CloudFront Distribution .png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_6cd_Updated_CloudFront_Distribution_.png)
        
    5. Change the **Viewer Protocol Policy** to **Redirect HTTP to HTTPS**
    6. Set **CNAME** to your primary domain name (**myexample.com**)
    7. Select the **custom SSL Certificate** earlier created with ACM
    8. Select a **Security policy**: **TLSv1.2_2021 (recommended)**
        
        ![Step 6bb_Create CloudFront Distribution .png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_6bb_Create_CloudFront_Distribution_.png)
        
    9. Set **default root object** for distribution to **index.html**
    
- **Create the Second Distribution**
    1. Replicate step 1 to step 5 as shown in the first distribution
    2. Set **CNAME** to your primary domain name (**www.myexample.com**)
    3. Replicate step 7 to step 9 as shown in the first distribution
        
        ![Step 6d_ CloudFront Distribution List.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_6d__CloudFront_Distribution_List.png)
        
    
- **Set up Error Page redirects to index.html**

This setting will redirect all error responses to index.html, letting the single-page application handle routing.

1. Navigate to the distribution **Error page tab.**
2. We will create two custom error responses (403 Forbidden and 404 Not Found). Click on **Create custom error response** 
    
    ![Step 6c_Set Up Redirects.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_6c_Set_Up_Redirects.png)
    
3. Set **HTTP Error Code** to **403/404**
    
    ![Step 6ca_403 Error Set Up Redirects.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_6ca_403_Error_Set_Up_Redirects.png)
    
4. Set **Error caching minimum TTL** to **0** seconds
5. Set **Customized error response** to **Yes**
6. Set **Response Page Path** to **/index.html**
7. Set **HTTP Response code** to **200: OK**
    
    ![Step 6cb_404 Error Set Up Redirects.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_6cb_404_Error_Set_Up_Redirects.png)
    
8. Repeat the above steps for the second distribution

### Step 7: Configure Route53 DNS record

This configuration points Route53 to our CloudFront distribution. We will create an **A Record**, which will serve as an alias that resolves our custom domain name to our CloudFront distribution. 

- Navigate to **Route 53**, then click on the created **hosted zone**
- Click on **Create record (Root Domain)**
    1. Keep the **record name** blank to create a record for your root domain (**myexample.com**)
    2. Set the **record type** to **A-IPv4 address**
    3. Set **Alias** to **Yes**
    4. Set **Alias target** to your **First CloudFront distribution** from the dropdown
    5. Click on **Create**
        
        ![Step 7a_Create A_Record for CloudFront.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_7a_Create_A_Record_for_CloudFront.png)
        
- Click on **Create record (Sub-Domain)**
    1. For the **record name** enter **www** to create a record for your sub-domain (**www.myexample.com**)
    2. Repeat steps 2 and 3 above 
    3. Set **Alias target** to your **Second CloudFront distribution** from the dropdown
    4. Click on **Create**
        
        ![Step 7a_Create A_Record for CloudFront 2.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_7a_Create_A_Record_for_CloudFront_2.png)
        

### Step 8: Modify S3 Bucket Policy

Lastly, we will navigate to our S3 bucket to remove the bucket policy added in Step 3 above. This modification will remove public access to our S3 bucket, and grants access to CloudFront only. 

![Step 7a_Update Bucket Policy.png](Hosting%20a%20Vue%20js%20Single%20Page%20Application%20(SPA)%20on%20%2008c4957b6a8e48cfb0191d8ba0795406/Step_7a_Update_Bucket_Policy.png)

**Note:** Making changes to your SPA can take some time to propagate. However, you can create a CloudFront Invalidation to clear the cache for CloudFront when testing new configurations.

That’s it for today, great work. Now, if you try accessing the URL, you should see that your site is live. Also, let’s check if the HTTPS configuration is working and if HTTP redirects to HTTPS.

### Conclusion

The goal of this tutorial was to guide us through the steps of deploying a Vue.js application to AWS for hosting and making the necessary changes to allow HTTPS and a custom domain name.

### **Thanks for reading!**

If you enjoyed reading this post, follow on **[Twitter](https://twitter.com/nigerian_boi)** and [**Medium**](https://medium.com/@oluwajuwonajana) for more content. Please, If you have any feedback, suggestions, or something you’d like to see in a future post. Kindly leave them in the comments below, and I’ll do my best to get back to you.
