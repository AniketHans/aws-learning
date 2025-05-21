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
18.
