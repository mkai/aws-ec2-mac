# AWS EC2 Mac

Spin up a Mac EC2 instance on AWS using CloudFormation and GitHub Actions

https://aws.amazon.com/de/ec2/instance-types/mac/

## How to deploy

- Fork the repository
- Add AWS credentials to repository secrets
- Run workflow "Deployment" and insert variables

### Example variables

- Instance type: e. g. `mac2.metal` (M1 Mac mini)
- AMI ID: e. g. `ami-00171e83b9ddc38bd` (macOS Sequoia 15.4.1)
- Region: for example `eu-west-1`
- Availability zone: for example `eu-west-1b`
- VPC ID: e. g. _vpc-0123456789abcdef_ (should be in the chosen AWS region, e. g. `eu-west-1`)
- Subnet ID: e. g. _subnet-0123456789abcdef_ (should be in the chosen availability zone, e. g. `eu-west-1b`)
- SSH key name: e. g. `my-keypair` (should already exist, see also _Prerequisites_ below)

## Cost

> [!WARNING]  
> The Mac dedicated host has a minimum allocation time of 24 hours, which means there is a **minimum cost** of approx. USD 15.60 for the host (depending on the host instance type) once it is running. The host cannot be destroyed before that time. After the 24 hours, the cost is caclulated per minute as usual.

## Using the instance

### SSH access

1. Download key pair

   From AWS Console, download the key pair used for the EC2 instance to your local machine.
   
   Ensure it has the proper permissions:
   
   ```shell
   chmod 400 my-keypair.pem
   ```

2. Connect to the instance:

   ```shell
   ssh -i my-keypair.pem ec2-user@<my-instance-hostname>.eu-west-1.compute.amazonaws.com
   ```

### VNC access

1. Enable macOS screen sharing

   ```shell
   sudo defaults write /var/db/launchd.db/com.apple.launchd/overrides.plist com.apple.screensharing -dict Disabled -bool false
   sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.screensharing.plist
   ```

2. Set password for `ec2-user`

   ```shell
   sudo /usr/bin/dscl . -passwd /Users/ec2-user
   ```

3. Open SSH tunnel

   ```shell
   ssh -i my-keypair.pem -L 5900:localhost:5900 ec2-user@<my-instance-hostname>.eu-west-1.compute.amazonaws.com
   ```

4. Connect to VNC

   On a Mac, for example, press Cmd+K in the Finder and connect to `vnc://localhost:5900`.

## Prerequisites

### Add AWS credentials to repo

* Add `AWS_ACCESS_KEY_ID` to repository variables.
* Add `AWS_SECRET_ACCESS_KEY` to repository secrets.

### VPC setup

In case you have not set up a VPC yet: _AWS Console_ > _VPC_ > _Your VPCs_ > _Actions_ > _Create default VPC_

> [!NOTE]  
> Make sure to create the VPC in the same region as your new EC2 instance.

### Create key pair (if needed)

```shell
aws ec2 create-key-pair --key-type rsa --key-name my-keypair --region <my region> --query 'KeyMaterial' --output text > my-keypair.pem
chmod 400 my-keypair.pem
```
