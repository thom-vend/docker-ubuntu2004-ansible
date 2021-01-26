# How to convert an ami to a docker image:

Run a fresh instance
```
aws --region ap-southeast-2 \
  ec2 run-instances \
  --image-id ami-0e2e14798f7a300a1 \
  --instance-type t3a.micro \
  --key-name thomas-test \
  --security-group-ids sg-9906c4ff \
  --subnet-id subnet-2db3fe74 \
  --instance-initiated-shutdown-behavior terminate \
  --associate-public-ip-address \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=thomas-ami-to-docker-img}]'
```

Output is like:
```
# OUtput:
#{
#    "Groups": [],
#    "Instances": [
#        {
#            "AmiLaunchIndex": 0,
#            "ImageId": "ami-0e2e14798f7a300a1",
#            "InstanceId": "i-07699f1054d09028d",
#            "InstanceType": "t3a.micro",
```

ssh on the instance
```
ssh -i ~/.ssh/thomas-test.pem ubuntu@3.26.57.52 'whoami'
```
Cleanup

```
ssh -i ~/.ssh/thomas-test.pem ubuntu@3.26.57.52 'apt-get remove -qy --purge snapd'
```

Download the tar ball:
```
ssh -i ~/.ssh/thomas-test.pem ubuntu@3.26.57.52 'sudo tar -C / --acls --xattrs --one-file-system -cz /' |cat - > ami-rootfs.tar.gz
```

Create the docker image from this tarball
```
FROM scratch
ADD ami-rootfs.tar.gz /
CMD ["/bin/bash"]
```

```
docker build -t ami-rootfs:latest .
```
