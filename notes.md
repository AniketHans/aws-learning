# AWS LEARNINGS

### EBS

1. EBS stands for Elastic Block Storage.
2. It acts as an extra storage to EC2 instances that we can mount on it.
3. If we create an EC2 instance, a new root volume is always created by default and it is attached to the EC2 instance. In it, the OS is installed and files related to it are stored.
4. The root volume will get deleted once the EC2 instance is terminated and the data is lost.
5. Thus, we can create a separate new EBS volume and mount it to the EC2 instance and after some config we can save our data to it.
6. This explicitly volume created can be prevented from deleting and can be mounted to the new instance in case of termination of the previous EC2 instance.
7. The EBS volumes can be of many types like General Purpose, IO1, IO2 etc based on the IOPS(Input-Output Operations Per Second) and throughput.
8. You can only increase the size of any existing volume.
9. Also, some EBS volume types can be mounted to multiple EC2 instances at the same time.
10. For mounting the EBS volume to any EC2 instance we need to create the EBS volume in the same region and availability zone as of the EC2 instance.
11. We cannot move the EBS volumes across aws regions but we can create a snapshot of the volume. This snapshot can be used to create similar EBS volume with its data in another region or availability zone
12. The snapshots taken are saved to S3 bucket. S3 is a region specific service means same S3 bucket can be accessed in different availability zones of the aws region. Thus, creating the EBS volume in another availability zone of the aws volume is easy.
13. Note: The EBS volume is an availability zone specific service.
14. The backups are taken in incremental strategy by default.  
    ![Backup strategies](./resources/images/backup-strategies.png)
15. Suppose you have taken a snapshot of 50 GB data stored presently in your volume. Now, if after one week, the data size increased to 60 GB then the new snapshot will be taken of the new 10 GB added.
16. Hence, there will be 2 snapshot, one containing 50 GB data info and the another containing 10 GB data.
17. Now, suppose the 50 GB snapshot got deleted then AWS in backend will transfer the 50 GB data to the 10 GB snapshot to prevent the data loss.
18. It means, before deleting any snapshot, the AWS will transfer its data to the immediate next snapshot to prevent data loss.
19. Note: The S3 bucket in which the sanpshot is stored is managed by AWS, we can't directly access that S3 bucket as it is not present in our AWS account directly.
20. The volume of the new EBS volume created from the snapshot should either be same or more than the volume of the EBS volume from which the sanpshot is taken.
21. The volume type can be changed while creating it from the snapshot.
22. Lifecycle manager can be used to automate the process of taking the snapshots on regular intervals or specfic time.
23. By default, if we delete a snapshot, it is deleted permanently. We can prevent this behaviour by creating a retention policy for the snaphots in `recycle bin` so that the deleted snapshots are stored in recycle bin for the set period before getting deleted properly.
24. We can copy the snapshot from one aws region to another by using the `copy snapshot` functionality.
25. We can encrypt a volume as well.
26. The snapshot of an encrypted volume is so encrypted. But we can create an encrypted snapshot from an unencrypted volume.

### AMIs

1. AMIs stands for Amazon Machine Image
2. These are images that can be used to spin up new EC2 machines with same config.
3. We can the image from an instance or from a snapshot.
4. The difference between using custom script and AMI is that if we use the custom script, the instance is first created and then the custom bash script is run on it. If we create the instance from the AMI, the instance will have the required config as soon as it gets spun up.
5. If we create an AMI from the EC2 instance, AWS first takes a snapshot of the instance and then creates an AMI from it.
6. By default, the AMIs created are private. But we can make them public to be used by anyone.
7. We can also share the AMI with any AWS account by providing its AWS account id.
8. For deleting the AMI, first deregister it. It will get deregistered from the snapshot it was created. Then, you can delete the associated snapshot.

### ELB

1. ELB stands for Elastic Load Balancer
2. ELB has multiple EC2 instances connected to it.
3. Instead of hitting the IP of the EC2 instance, we hit the Load balancer's ip and it distributes the load amoung the EC2 insatnces connected to it.
4. The ELB checks for the health of the EC2 instances connected to it using the `status` api that we generally create in our REST applications. We need to configure the health check mechanism while setting up the ELB
5. We get a DNS name after setting up the ELB. We dont get any ip address of the ELB. The DNS name can be used to access the application from any of the EC2 instances.
6. If you enable HTTP traffic from anywhere in all the EC2 instances then the EC2 instances can be accessed both from the ELB as well as outside world. Thus, we need to configure the security groups of the EC2 instances to allow HTTP traffic comming only from the ELB and not from the outside world.
7. To configure the security group of the EC2 instance for the ELB, we can set the type: HTTP, protocol: TCP and source: `<Security Group of the ELB>`. It means, suppose `sg-ELB` is the security group that you created for the ELB and attached it to the ELB. Then, while configuring the HTTP inbound rule of the security group, for the EC2 instances under the ELB, set the source of HTTP as `sg-ELB`. This config will only allow the Ec2 instances to be accessed from the ELB and not the outside world.
8. Application Load Balancer (ALB)
   1. It works on the Application Layer of the OSI model.
   2. We can create some EC2 instances that we want to keep behind the ALB. Here, we need to create some target groups and assign the created EC2 instances to any of the target group. We define some rules in the target group. The rules define the type/category of the traffic. After getting the type of traffic, the ALB reroutes the traffic to the EC2 instances of the apt target group.
   3. The ALB works on the application layer. Thus, it has all the information of the HTTP request that the client is sending to EC2 instances underneath the ALB. Suppose, we have 2 routes /home and /about in our REST application running in the EC2 instances. We want the /home requests to route to some set of EC2 instances and /about requests to another set of EC2 instances. For this, we create target groups and assign some EC2 instances to it.
   4. We can add/edit the rules, to reroute the requests for apt target group EC2 instances, in Listeners section of the ALB. There, we can set if the request has /home as the path then reroute to the traffic to the EC2 instances of a particular target group and similarly for the /about path.
   5. Some use cases are, if we have a domain `www.abc.com`. If the request comes to `www.abc.com` then we can reroute this to our front application servers/EC2 instances and if the request comes to `www.abc.com/api/` then we can reroute the request to our backend servers/EC2 instances.
   6. Similary, we can use query parameters in the request to decide the target Ec2 instances.
   7. We have the property called Group level Stickiness, which is disabled by default. Suppose I have 2 Ec2 instances (E1 and E2) under the target group. Let a client send requests, then my ALB can either send the request to E1 or E2 based on the load. Now, if the same client again requests, then again my ALB can send request to either of E1 and E2 if the Group level Stickiness is disabled. But if Group level Stickiness is enabled, then ALb will direct all the requests from same client to same EC2 instance either E1 or E2 each time. This scenerio is useful in case of stateful services where the state of the client is saved.
   8. The stickiness can be achieved by using cookies. We can use Load Balancer generated cookies as well. In this case, in request headers, you will see the aws load balancer generated cookie being attached to identify the request.
   9. Cross Zone load balancing is enabled by default in ALB. It means, if we have multiple EC2 instances from different availability zones attached to a load balancer, then the load will be distributed evently amoung all the EC2 instances irrecpective of the availability zone.
   10. If Cross zone load balancing is disabled, then the traffic will only be distributed amoung the EC2 instances that are in same availability zone as that of the load balancer.
9. Network Load Balancer (NLB)
   1. It works on the transport layer.
   2. It mainly has info about the source and destination IP address along the ports. Network load balancer does not have info about the data that is being communicated between source and destination.
   3. As NLB does not involve processing of data so it is faster than ALB.
   4. Here, the target groups are also created. We divide the target groups based on the protocol and port of the application. It means instances belonging to a target grp can be used to serve the application at port say 4569 and other EC2 instances belonging to the another target grp can can be used to server another application at port say 6708
   5. Cross zone load balancing is disabled in NLB

### ASG

1. It stands for Auto Scaling Group
2. Auto scaling is used when the actual traffic on the instances is very different from the predicted traffic. Either, the traffic is very huge and our current number of instances are not able to handle them or the traffic is very less and we have some extra instances running.
3. We can have 2 types of scaling:
   1. Vertical scaling
      1. Here, the size of the instance is increased to incorporate the increased demand
      2. If you are using a paid operating system, then its better to go with vertical scaling.
      3. If you have database running on the instance, then go with vertical scaling.
   2. Horizontal scaling
      1. Here, the number of instances under the load balancer are increased.
4. You can create a launch template for your EC2 instance. A launch template is a blueprint with some configurations that can be used to create EC2 instances with the given config. All the EC2 instances created from the template are identical.
5. We set the desired, minimum and maximum number of instances needed in the ASG. ASG will act as an orchestrator and will maintain the desired instances in the system and as soon as the traffic increases, it starts increasing the number of instances.
6. As soon as the ASG is deleted, the instances created by the ASG will also gets deleted from the system.
7. We can have the following types of scaling:
   1. Scheduled actions
      1. This is used when you know the events in the future where the load will be high and more instances will be needed.
      2. The event can be on specific date and time or periodic as well.
   2. Predictive actions
      1. This can be used with aws' predictive algo where through the past data, aws analys when and how many new instances will be required to meet the potential demand.
   3. Dynamic actions
      1. It has the follwoing scaling policies:
         1. Simple scaling:-
            1. Here, we need to create a cloudwatch alarm for anything like CPU utilization etc. When the alarm is triggered, based on the config new instance will gets created
            2. We can either increase or decrease the number of instances
         2. Step scaling
            1. It is similar to Simple scaling except here we can config increasing and decreasing of instances based on multiple cloudwatch alarms
         3. Target Tracking scaling
            1. Here, we dont need to create cloudwatch alarm manually. It also uses the cloudwatch at the backend but we only need to provide the condidtion which will trigged the scaling on instances like say, when average CPU utlization of all the instances becomes more than 50% then start creating new instances
8. When you create the ASG for a load balancer, it asks for info about existing target grp or creating target grp. After this, the addition or removing of scaled instances into the target group will be taken care by the AWS itself.

### IAM

1. Identity Access Management
2. IAM is a global service. It means its config will remain available in all the avalibility zones.
3. It is used to give specific roles and permission to users. We grant some aws services permissions to the user so that the user can perform some actions on them only.
4. We attach policies to the users. A policy is a document in which the aws services and their permissions are written.
5. We can create groups with some policies attached to it. Later, we can add/remove users from it.
6. Suppose, there are 2 policies:- S3 full access and S3 no access. Both of these are attached to a user. Then, the user will not be able to access the S3. In AWS Identity and Access Management (IAM), when a user is associated with multiple groups, the permissions are cumulative. This means that if you attach a user to two groups—one with permissions to read and write in Amazon S3, and another with explicit denials to read and write in S3—the user will ultimately be denied those permissions. In IAM, explicit denials take precedence over permissions granted. This is known as the "deny overrides" principle. So, if any group the user belongs to denies a certain permission, even if the other groups grant that permission, the denial will take precedence and the user will be denied access. In your scenario:
   1. Group 1 grants read and write permissions to S3.
   2. Group 2 denies read and write permissions to S3.
      If you attach both groups to a user, the user will not have permissions to read and write in S3. The explicit denial from Group 2 will override the permissions from Group 1. So, the user will essentially have the least permissive set of permissions across all the groups they are associated with. In this case, the user would not be able to perform any S3 actions due to the explicit denial in Group 2.
7. If you copy policies from a user to another. Then, if the source user is a member of a group and you copy its permissions to the other user, then the other user will also become the member of the group that the source user is.
8. Roles in AWS:
   1. Roles are attached to an AWS resource.
   2. The role can give access to an aws resource to access another aws resource mentioned in the role policy.
   3. Like, we can create a role with a policy to access the S3 and assign that role to EC2 instance or lambda. Thus, now we can access the S3 from our EC2 instances or lambda.
   4. Similary, we can assign the role to access the secrets manager to our lambdas.
9. Cloudshell
   1. It is an AWS cli that can be accessed on the web from the aws account.
   2. The account through which the cloudshell is accessed will already be configured in the cloushell
10. Thus, we can access the AWS services through 3 ways:
    1. Configure the aws cli in our local system
       1. Install aws cli from web
       2. In terminal, run `aws configure --profile <name>`
       3. This will ask for your secrets
       4. then using, `aws help` you can further access the aws services like S3, ec2 etc
    2. Providing specific service role to EC2 and lambda
       1. The role can be assigned to EC2 instance and then the aws services can be accessed by installing the aws cli in the ec2 instance
       2. We dont have to configure the credential as the role is assigned to the Ec2 instance
    3. Using Cloudshell
       1. We dont need to install anything as this can be accessed from the aws console.

### S3

1. S3 stands for Simple Storage Service
2. Here, we can storage our data.
3. This is an aws core service.
4. The bucket can be created in a region. The bucket name should be unique across all the buckets present in aws.
5. The files uploaded in S3 bucket are refered to as objects.
6. S3 bucket dont support creating folders. Suppose you created a folder, with name f1, and added a file file1.txt in it. Then, the folder names will only be added as prefix to the file name and hence the final name will be f1/file1.txt Here, `f1/` is the prefix for file `file1.txt`
7. Versioning can be enabled to prevent losing of an object if it is overridden by another object with the same name. If versioning is enabled on the bucket and some object is deleted by mistake then you can retrieve it by foloowing steps:
   1. First enable the show versions toggle
   2. The deleted file will have a type as `Delete marker`
   3. Delete this file with type `Delete marker` and your mistakenly deleted file will be retrieved.
8. If you have enabled versioning for a bucket then you can't disable it. you can only suspend the versioning. It means the files, which were uploaded when the versioning was enabled, will still show versions after the versioning suspension. But the newly uploaded files will not have any versions.
9. You can use S3 to serve static pages of your website. It means the pages will not have any api calls etc, just a file containing html, css and js. You need to enable the static website hosting setting in the S3 bucket.
10. We need to make both the bucket and the objects in the bucket public for them to be accessed publically.
11. In case of static hosting, we can redirect one bucket to another. It means suppose we have redirection from bucketA to bucketB with both having static website hosting enabled, then if we try to access the bucketA through its hostname, it will redirect us to the bucketB's hostname.
12. We can set redirection rules as well in static website hosting buckets.
13. S3 Accelaration
    1. Suppose you have a bucket in aws virginia region and you live in India, you want to upload some files to your bucket.
    2. Normally, it will take a lot of time as you will have to upload you file directly to the bucket which is very far from your location
    3. S3 accelaration can be enabled to cut short the time taken to upload the file.
    4. In aws, we have lots of edge locations like mumbai, hyderabad, delhi etc in ap-south-1.
    5. In S3 accelaration, say you are in India, what we do is we upload the file to our nearest edge location and then aws itself will transfer it, using its own network, to your bucket in virginia. AWS is smart enough to know if direct uploading will be faster or accelarated uploading and uses the faster one.
14. Same Region Replication (SRR) and Cross Region Replication(CRR)
    1. The objects in a bucket are copied to mutiple availability zones by default for fault tolerance
    2. When the object is copied in the same region, we call it SRR
    3. When the object is copied in multiple regions, we call it CRR
    4. You need to set the replication rules in a bucket, where you will be defining the source and the destination bucket. The destination bucket will be the one where the backup of the source bucket will be kept. The dest bucket may be in same region or other.
    5. Note, the replication will not replicate the delete marker until explicitly checked.
15. S3 storage classes
    1. We have multiple storage classes based on the frequency of object retrieval, availability of object.
    2. We can select the storage classes only at the object level not at the bucket level. It means while uploading the object in S3 we can select what storage class like standard, glacier etc the object belongs. The storage class then will decide the fequency, speed etc of the object retrieval.
16. S3 lifecycle management
    1. We can define the lifecyle of an object present in our bucket using the lifecyle rules.
    2. It means suppose an object is there which belongs to the S3 standard storage class. Now, you want the object to move to S3 Glacier storage class after some time, say 30 days and after 90 days you want to delete the object. You can do all this my setting up the lifecycle rules in bucket.
17. CORS:
    1. Suppose, you have a website www.web1.com and you have some images, html files etc stored in S3 bucket.
    2. Now, you want to use the resources of S3 bucket into your website. But as you can see the origin of your website will be www.web1.com and the origin of S3 will be, lets say, s3:youbucket.aws.com.
    3. Now since the origin of both the things are different, you cant use the resources of s3 directly into the web1.com.
    4. For this to work, you need to enable CORS and whitelist the web1.com url in your S3 config so that the web1.com can access the content of S3 bucket.
    5. Note, the CORS by default blocks the external websites to access your website's data, means only the requests from same domain will be catered and rest will be denied.
    6. In case of S3 say there are 2 buckets, bucket1 and bucket2. Both the buckets have static website hosting enabled. Now, say you have your index.html file present in bucket1 and the other file say content.html in bucket2. Suppose, in the index.html, we are trying to access the content.html using jquery.
    7. Now, since the CORS policy is not set by default so you are going to get CORS error. For, bucket1 to content of bucket2, you need to enable/whitelist the bucket1 in bucket2's CORS policy.
18. Presigned URLs
    1. Suppose you have a S3 object which you want to share with someone outside your org. You can't make the object public as that can make the object accessible to anybody.
    2. Presigned urls are urls which allow someone to access any object for a duration of time and after that the url will get expired and no one will be able to access the object with that url.
    3. In AWS, select the object and go to object actions. After that select generate presigned url with a time limit.
    4. This helps in sharing object with external people without making it public.
19. S3 encryption
    1. You can encrypt the bucket objects either at the client side (at the application level, before storing to S3) or at the server side (means at the AWS side)
    2. For server side encryption, you can use either S3 managed encryption key or key from Aws Key Managment Service.
    3. You can have a particular encryption mechanism at the bucket level. Also, you can override the bucket level encrytion mechanism for any object with a different encryption mechanism.

### Cloudfront

1. Suppose you have a server in US and you are in India, say Moradabad. Now, you want to access a webpage hosted on the server. Now, ideally you should directly request the US server and fetch the webpage. But since the distance between the client and the server is too large, then the request will take some time to fulfill.
2. As we know, there are many aws edge location. Now, suppose there is a edge location with aws servers in Delhi, India and I am in Moradabad. Now, if I want to request a webpage from US server, I will make a request to my nearest edge location i.e Delhi. The request will reach the Delhi servers and then AWS will use its own network to communicate the Delhi location to the US location for the requested webpage. Since, AWS is using its own network, the request turnaround time will be very less. Then, after retrieving the webpage in Delhi location, AWS will send the page to client and also keep a copy of the webpage for itself in case of another request for the same page. The webpage will be stored in AWS cloudfront.
3. Cloudfront is like a cache which can store webpages, files etc. at a edge location
4. Cloudfront can store dynmic as well as static content.
5. Cloudfront keeps sending the response back to the client as soon as it starting getting response from the server. It means if a file is requested by user which is very large in size, Cloudfront will keep sending the file to the user for download as soon as the first bit of the file is recieved at the cloudfront(at the edge location) from server.
6. Thus, cloudfront can be used for streaming as well because the client will request for the data to an edge location, rest is taken care by the AWS network to ask for the data from far server and then sending the data back to client as soon as it is recieved at the edge location.
7. Even if the server is sending the data (large data) in multiple packages/chunks, the cloudfront will send them to the user as soon as it recieves them so that the latency can be reduced. Thus, cloudfront is good for streaming purpose as well.
8. Once you setup a cloudfront configuration, it will be deployed at all the aws edge locations.
9. Also, while creating the cloudfront config, you can add a custom header to the request going from cloudfront to your server. It means the client will request for webpage/file to the nearest cloudfront and then the cloudfront will add that custom header in the request and send it to the server. The custom header can be checked at the server to know if the request is comming from the cloudfront or not by cross checking the custom header that you added. This can be used to filter out the request comming from cloudfront.
10. If you have updated the webpage at your server but your cloudfront is still showing old webpage then you can create cloudfront invalidations and invalidate the data for the path you want. Next time you fetch the webpage, it will be an updated one.
11. Enabling only cloudfront to access your Load Balancer or EC2 instance:
    1. If you want nobody except the cloudfront to access your Load balancer and EC2 instance then you can do this by enabling the IPs that are allocated to cloudfront only in the load balancer/EC2 instance's security group.
    2. AWS maintains some list of IPs itself like the list of IPs of cloudfront
    3. You can get the cloudfront list by Going to `VPC ---> Managed Prefix lists ---> amazon global cloudfront list`
    4. Copy the id of the cloudfront list
    5. Now, add the id to the source http and https in the security group of the load balancer or EC2 instance
    6. After this, the load balancer or EC2 instance will only get hit from cloudfront.
12. You can also access a private S3 bucket's object using cloudfront. All you need is to create a cloudfront distribution for that S3 bucket and change policy of the S3 bucket to enable cloudfront to access the objects of the bucket.
13. Cloudfront path based routing
    1. Suppose you have a requirement to create a cloudfront distribution where if /images path is hit, we need to get retreive data from S3 and if /web is hit, we need to access our EC2 instance.
    2. You can do this my adding multiple cloudfront origins with the paths mentioned above.
    3. At the time of creation of cloudfront distribution, you can only add one origin. But after its deployment, you can add multiple origins like S3, EC2 etc.
    4. After this, go to behaviours in that cloudfront distribution, add the path and the origin you want to call for that path.
    5. This will make the desired changes.
14. You can also set a custom error page for different error codes like 404, 500 etc.
15. You can also put geographic restrictions like allowing or blocking access to the cloudfront from any number of countries.
16. Note: any changes to the cloudfront config will be deployed to all the edge locations so things might take time to deploy.
17. Usages of cloudfront
    1. You can only allow request from cloudfront to call apis by adding custom header to the req and then checking that req header in a middleware.
    2. You can also set the TTL for api response in cloudfront by adding `cache-control` response header from the server in the response.
    3. Suppose, you have the following 2 urls hit one after the other:
       1. `cloudfront-url.aws.com/user?page=1&size=5`
       2. `cloudfront-url.aws.com/user?page=2&size=6`
       3. It means we have query params along with path.
       4. Cloundfront is a cache and by default it puts only the path as key and the response as value. It means it does not take the query params into the consideration of key.
       5. Thus, both the above mentioned urls will return the same response as the path is same. The first one will be a cache miss and other one will be a cache hit. This is because the query params in not taken into consideration in the cache key hence both the requests will be considered as same.
       6. We can enable the query params consideration by enabling Query strings setting in cache policy. After this, the query params will also be considered in the cache key and hence the response will be different for the above requests
       7. Similary, we can enable header and cookies as the part of the key in cache. This can be helpful for authorized users.

### Amazon VPC

1. VPC stands for Virtual Private Cloud. This is AWS' networking service
2. All the services that you deploy need to communicate with each other thus there is need for networking
3. VPC has the scope in a region only.
4. Creating a VPC
   1. We need to provide range of private IPs that will be used in the VPC
   2. The network created using VPC is a private network so private IPs are needed. You can search on the internet about private IP range
   3. You need to provide the IPv4 CIDR. Now, suppose you have chosen the private network intial IP as 192.168.0.0 , now you thought you are going to have not more that 2^16 servers for your services. Then what you can do is you can put the 192.168 as static prefix for the server IPs and have the variable IP from 0.0 to 255.255, Hence the full range of IPs for your servers will be 192.168.0.0 to 192.168.255.255. Since the first 16 bits of your private IPs will be static so the CIDR for you VPC will be `192.168.0.0/16`
   4. Here above the network id will be 192.168 since this is static
   5. The created VPC will be available in all the availability zones under the region
5. Subnets
   1. We can divide the bigger network, where devices can have IPs ranging from 192.168.0.0 to 192.168.255.255, into smaller subnetworks or subnets.
   2. In a college, we have multiple networks may be divided based on departments. Although, the college will have a VPC but there can be multiple subnets for multiple departments.
   3. Devices connected to a subnet can communicate with each other seemlessly
   4. Now, suppose we want to create 4 subnets from the above VPC and lets have their IP ranges as follows:
      1. Subnet 1: 192.168.1.0 - 192.168.1.255, thus IPv4 CIDR will be 192.168.1.0/24 since first 24 bits are static for the subnet
      2. Subnet 2: 192.168.2.0 - 192.168.2.255, thus IPv4 CIDR will be 192.168.2.0/24 since first 24 bits are static for the subnet
      3. Subnet 3: 192.168.3.0 - 192.168.3.255, thus IPv4 CIDR will be 192.168.3.0/24 since first 24 bits are static for the subnet
      4. Subnet 4: 192.168.4.0 - 192.168.4.255, thus IPv4 CIDR will be 192.168.4.0/24 since first 24 bits are static for the subnet
   5. Subnet's scope is in a availability zone only
6. Note: Whenever we create a security group, we select a VPC. Now if we create another VPC and use it for our instances, we will not get that security group as option while attaching it to the EC2 instance because the security group is attached to some other VPC.
7. If you associate you vpc or subnets to your servers, then your servers will be able to communicate with each other but you will not be to access them using CLI, API etc because there is no internet access to the vpc.
8. To provide the internet access to your VPC, you need to create an `internet gateway` and attach it to you VPC.
9. Once you create a VPC, a routing table gets created and get attached to it. Routing table defines what traffic should be allowed and disallowed and what IPs are local and what are for the internet. For our setting, any machine with the private IP as 192.168.0.0 - 192.168.255.255 will be considered as local and its traffic will be local used to communicate with other server in the same IP range.
10. For us to access the servers, attached to our VPC, through internet, we need to attach IP range 0.0.0.0/0 with target as Internet gateway, that we created above, to the routing table attached to our VPC.
11. As we have created 4 subnets in our VPC and by default a routing table gets attached to our VPC, so the same routing table will gets attached to all the 4 subnets. It means any changes in the routing table will be applicable to all the subnets. It means if internet access is introduced in the routing table then all the subnets will have internet access.
12. The subnet which has internet access is called Public subnet and which does not have internet access is called Private subnet.
13. We can have our EC2 instances, where we have our application running, in the public subnet and the intances, where we have databases, in the private subnet so the application servers can be accessed by the internet but the database server can only be accessed by the application instances which are part of the same VPC and has the private ip range between 192.168.0.0 to 192.168.255.255.
14. In private subnet, the servers, with the defined IP range in the VPC, can communicate with other and nobody from outside.
15. Create public subnet and private subnet with a VPC:
    1. There is one default routing table attached to all the subnets under a VPC and it does not have public internet access enabled by default
    2. You can create a new routing table under the same VPC and attach some of the subnets, which you wanna keep private, to the routine table. The newly created routine table will not have internet access by default.
    3. You can provide the internet access to the default created routing table so the subnets that are still attached to it will get internet access and become member of public subnets.
    4. In this way, we can create public and private subnets within the same VPC
16. The instances created under any public subnet can be accessed by internet and the instances created under the private subnet will not be accessed by internet but only by the servers belonging to the subnets under the VPC
17. You can also put your application server in private subnet and to access the application, you can create a proxy server in the public subnet. So the user will only be able to request your proxy and then the proxy will forward the request to application server as both belong to the same VPC.
18. Suppose, you have 2 subnets, S1-public and S2-private which are public and private respectively. Now, say you have 2 instances I1 and I2 connected to S1-public and S2-private respectively. If you try to ssh the I1, you will be able to do it since the I1 has the internet access and if you try to ssh the I2, you will not be able to do it as it does not have internet access. Now, you want to install something in the I2 using terminal but from your system you wont be able to do it. So what you can do is, first ssh into I1 and from I1, ssh into I2. I1 and I2 will be able to communicate with each other as both of them belong to the same VPC. After that, you will be able to install things in I2. Here, I1 acted as a `jump server` as it is here used to access the I2.
19. Note: Even after ssh to I2 from I1, you wont be able to access internet through I2 as it is part of private subnet and does not have access to internet.
