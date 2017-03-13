# Lab 00

## Connect to Amazon AWS servers

Use the chmod command to make sure that your private key file isn't publicly viewable.
```
chmod 400 /path/Academy.pem
```

Login to your server via SSH.
```
ssh -i ~/.sshall/Academy.pem centos@server_name.compute.amazonaws.com
```

Servers available:

* ec2-54-154-153-48.eu-west-1.compute.amazonaws.com
* ec2-54-154-2-68.eu-west-1.compute.amazonaws.com
* ec2-54-171-168-78.eu-west-1.compute.amazonaws.com
* ec2-54-154-116-10.eu-west-1.compute.amazonaws.com
