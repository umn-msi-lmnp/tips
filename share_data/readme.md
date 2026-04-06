# Share Data Outside UMN

Sometimes a PI asks us to make large raw data files available to outside collaborators. One way to do this is to create a bucket on Tier 2 for the data, put data there, and create a shareable link for the PI to share with collaborators. 

Original credit to Todd and Allison 

## Set up

Need to add: making sure aws config file is set up correctly

## Share Data

I tar and zip a folder with the data I want to share before copying it to Tier 2. 

```
# get md5sum
md5sum DehmProject100_Stat5.tar.gz 
ccad2453440544a26833fcda6d8fd06b  DehmProject100_Stat5.tar.gz


# create bucket on tier2
s3cmd mb s3://dehms-project100-fastq-files

# put data in bucket
s3cmd put DehmProject100_Stat5.tar.gz s3://dehms-project100-fastq-files

# check bucket contents 
s3cmd ls s3://dehms-project100-fastq-files

# get presign link
# max sharable time is one week
/projects/standard/lmnp/oconnorc/software/aws/v2/2.4.8/dist/aws --endpoint-url="https://s3.msi.umn.edu" s3 presign s3://dehms-project100-fastq-files/DehmProject100_Stat5.tar.gz --expires-in 604800
# output link: 
https://s3.msi.umn.edu/dehms-project100-fastq-files/DehmProject100_Stat5.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=13CQAOB3JYE5Q5QF4S8Y%2F20260406%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20260406T160518Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=7b8060c03ef7c9aa13e604e44205a8f42ce694bc1bc91ff

# how to get access to files 
wget -O 'DehmProject100_Stat5.tar.gz' 'https://s3.msi.umn.edu/dehms-project100-fastq-files/DehmProject100_Stat5.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=13CQAOB3JYE5Q5QF4S8Y%2F20260406%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20260406T160518Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=7b8060c03ef7c9aa13e604e44205a8f42ce694bc1bc91ff'

```

--expires-in is in seconds. 604800 is one week and the max time allowed. 
The above commands will not work because I changed the output link a bit, however it is a very long link. 

## Share in Globus

Coming Soon
