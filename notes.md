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
