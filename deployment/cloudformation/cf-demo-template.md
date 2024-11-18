---
icon: display-code
---

# CF Demo Template

1. EC2 only template
   1. ```bash
      ImageId: ami-0658158d7ba8fd573 //From AMI catalog
      ```
2. Launch in CF
3. Update by adding SG and Parameters manually via Edit in Infrastructure Composer + Validate

```yaml
Description: EC2 + SG Simple CloudFormation Demo.
Parameters:
  SSHLocation:
    Description: The IP range that can be used to SSH into EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.

Resources:
  MyWebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0658158d7ba8fd573
      InstanceType: t3.micro
      KeyName: ec2-keys-demo
      Tags:
        - Key: Name
          Value: WebSever

      SecurityGroups:
        - !Ref WebServerSecurityGroup
        - !Ref SSHOnlySecurityGroup

  SSHOnlySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22 from anywhere
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref SSHLocation

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0
```

4.  Pick up Instance public IPv4 and ssh into it&#x20;

    ```bash
    ssh -i path/to/AccessKey.pem ec2-user@13.52.125.28
    # check SG work
    curl http://localhost
    ```
5. It will <mark style="color:red;">**not work in browser**</mark> :exclamation: as Application Not Running
6. Install and Start

```bash
sudo yum install httpd
sudo systemctl start httpd  
sudo netstat -tuln
```

7. Verify it runs -> Look for `0.0.0.0:80` under `Local Address` with the state `LISTEN`.

```bash
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN

```

8. Curl localhost and output must be \<html> or open from Console&#x20;

