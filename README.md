Creating mongoDB cluster along with AWS infrastructure,


Ansible has more than 170 modules just for AWS related services, and many more modules for other cloud providers, we can easily create our infra with ansible, 

we can create VPC, subnet , security groups and what not, with the help of ansible, 

anisble also gives the feature of dynamic inventory, in this we dont have to provide the static host file, these dynamic inventories are generally useful in the cases where our servers keeps on changing in no.  and it is difficult to keep record of every newly created or newly destroyed server.

In our case, we are using ansible to create a VPC , 2 subnets for high availibility , a security group and a .pem keys to access our 7 newly created instances, we are writing a role to create this, 
we are creating a security group, and it will be automatically attached to the servers, 

our role will also be able to create a host file for these new resources, it will also put some other important varibles to the group_vars files of the respective groups.
It will create a .pem keys and will store it in the keys folder in /etc/ansible, keys will be used to access the newly created servers.
In our case we are taking these varaibles from the users,
VPC CIDR{10.0.0.0/24}
subnet A CIDR{10.0.0.0/28}
subnet B CIDR{10.0.0.16/28}
region {ap-south-1}
AMI – Ubuntu{default}
key-name – name of the .pem keys
security_group – the name of the security group
type-t2.micro{default}
these values are mentioned in the defaults/main.yml, user has the option to change these values, by passing –extra-vars

please note that we are just creating a fully functional AWS infrastructure with basic configuration, user will always have the option to change it through AWS consle, and i am creating this infrastructure with respect to the mongo cluster, all the configuration part is done by keeping the configuration of mongocluster in mind, that is why, we only have 27017-27020 and 22 port defined in our ingress,

noW, we are done with the infrastructure part ,in order to connect our mongo servers to create a cluster we need private_ips of all these servers, so we are  collecting the private_ips of all these servers and storing it in group_vars of the respctive groups, so that when we call our mongo role, it will get the required varibles from the group_vars files, 



now our infrastrucutre is ready, we have to install python on these servers to make them asseccible through ansible, coz ansible has only 1 dependency, i.e, the host servers must have python installed on them, as ansible requires python to interpret the modules, 
so by default, ansible provides some modules which doesnot needs python , these modules work on ssh, 
raw module
script module
now we will install python on all our servers, with the help of raw module,

now our servers are ready, we can start the provisioning part, 

with the help of mongo role we will create our mongo sharded cluster, which comprises of 
1 mongos server
1 config server – 2 replica set
2 shard server – 2 replica sets
 all the configuration files are stored in template section of the role, we will pass the values for the various variables defined inside the template file, 
for eg. We will pass port 27017 for the mongos server and 27018 for the shard server in the mongod.conf file,
after sending these config files to their respective server we will run the various mongo commands in order to create the cluster, in our case we are creating a bash script, and adding lines to it, while running a role, and after that running a script to complete the task, 
we will add shard and create replica sets , and we will attach these config servers and shard servers to our mongos instance
in order to verify the functionality of the server, we can run status command, 

mongoDB is one of the most popular NoSQL databases, it supports the option of sharding and replica sets for high availibilty and auto scaling, to make the full utilization out of it, we create a mongo cluster having mongos, config and shard servers having replica sets in each of them, 
shards are basically the collection of data, which gets equally distibuted across the  various shard servers, the distribution is decided by the config servers in mongo cluster,

and, replica sets are the replica of the data collection across the servers, which is useful in case of high availibility and in case of failure of any servers, we can retrieve the data fron the other server, there is also the concept of primary-secondary in replica set, mongo itself has the mechanism to decide the status of the servers, and in case if the primary server goes down, one of the secondary server is elected as the primary server, and that too is decided by the mongo itself,

in our case we are also creating a backup script, this script will run on the mongos server and take the backup of the whole cluster once in a while, 
this script will also create a tar file out of the backup file, and tar file will always have the unique name every time it is created, this tar file will also be stored to the AWS S3 bucket, through S3CMD, we just have to provide the AWS access keys and the AWS security keys, to access the AWS resources, 
so everytime the script will run. It will create a backup file and store it in the AWS S3 bucket, 
and we are also adding a cron entry for the script, so that the script will run as per requirement of the user, 


