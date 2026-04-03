# Accessing an API Gateway Private REST API via VPN

This is a demonstration of accessing an API Gateway Private REST API via VPN.

We're going to creat a Private API Gateway REST API and stick it behind an Internal (non-Internet-Facing) Application Load Balancer (ALB).

ALBs have DNS Names - and their IP addresses can change at any time.

To have a static, un-changing internal IP address for this, we're going to stick an NLB in front of the ALB.

We'll also setup a Client VPN connection, so we can demonstrate accessing this over a VPN connection (using the internal IP of the NLB)

It's going to look like:

![API Gw behind ALB and NLB](docs/API%20GW%20via%20NLB%20AWS%20Client%20VPN.drawio.png)

## Pre-Requisites

* Route 53 Hosted Zone (Public) - Somehow you need name resolution.  For demonstration purposes this is easiest, but what is actually required will depend on how your VPN connectivity is actually configured.

## AWS Client VPN Setup

First, we'll setup a VPC with public and private subnets, and an AWS Client VPN connection.

Relevant files are in the vpn folder.

1. Using CloudShell, clone this git repo
2. Run the easy-rsa.sh to create VPN Server and Client certificates and import into ACM
```
cd vpn
./easy-rsa.sh
```
2. Create the VPN stack.  Name it "myvpn"
3. Using CloudShell, run the after-stack-config.sh (cd into vpn folder)
```
./after-stack-config.sh
```
4. Download or otherwise copy the client-config.ovpn file and add a Profile in the AWS VPN Client
4. Create the other stack(s) (S3, API Gateway, AppSync, AppSync ALB )

Note that there will be a PUBLIC DNS record with a PRIVATE 10.* IP for the ALB.  This is convenient for this TECH DEMO, not a best practice for production.

Connect to VPN and then try to access one of the resources (for example the API Gateway via):

curl https://whateverdomainyouused.com/prod/data 

This should return a JSON output when connected to VPN, and the name will be unresolvable / unreachable if not connected to VPN

## VPN Cleanup

1. Delete the ALB/NLB/API Gw stack
2. Delete the VPN stack
3. Manually delete the ACM certs that were imported during VPN setup.
4. OPTIONAL: Delete the certificate files created for VPN 


# API Gateway Private REST API over ALB behind NLB

The NLB gives us STATIC, un-changing INTERNAL IPs.

The ALB will rewrite the HostHeader so the API Gateway service makes the connection to the Private REST API to be used.


Simply create the stack using apigw-over-alb-behind-nlb.yaml

Create a HOSTS entry to point yourcustomdomainname.com to a static IP of the NLB. OR just use the public hosted record (Not for production).


... and then curl (while connected to VPN, of course) :
```
curl https://yourcustomdomainname.com/prod/data
```
or
```
curl https://yourcustomdomainname.com/prod/healthcheck
```

The ALB's Target Groups have the IPs of the ENIs associated with the Interface Endpoints for the API Gateway service:
![API GW over ALB](docs/API%20GW%20via%20AWS%20Client%20VPN.drawio.png)

