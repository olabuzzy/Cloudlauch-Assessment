# CloudLaunch AWS Assessment

## Task 1: Static Website Hosting Using S3 + IAM
- Created 3 S3 buckets:
  - *cloudlauch-site-bucket* → hosts the static website (index.html, styles.css, etc.).
  - *cloudlauch-private-bucket* → private bucket, only accessible via IAM user with GetObject/PutObject permissions.
  - *cloudlauch-visible-only-bucket* → bucket that can be listed but objects are not accessible.
- Enabled *static website hosting* on cloudlauch-site-bucket.
- Attached a *bucket policy* to allow public GetObject access.
- Created IAM user cloudlauch-user with custom JSON policy.
- Restricted permissions:
  - Can only list all three buckets.
  - Can GetObject/PutObject in cloudlauch-private-bucket.
  - Can GetObject in cloudlauch-site-bucket.
  - Cannot delete objects.
  - Cannot read cloudlauch-visible-only-bucket contents.

### Static Website Link
👉 http://cloudlauch-site-bucket.s3-website.eu-west-2.amazonaws.com  


## Task 2: VPC Design
- Created VPC cloudlauch-vpc with CIDR 10.0.0.0/16.
- Subnets:
  - *10.0.1.0/24* → Public subnet (for future load balancer).
  - *10.0.2.0/24* → Application subnet (private).
  - *10.0.3.0/28* → Database subnet (private).
- Internet Gateway: cloudlauch-igw attached to the VPC.
- Route Tables:
  - cloudlauch-public-rt → routes 0.0.0.0/0 to IGW, associated with public subnet.
  - cloudlauch-app-rt → private, no internet route.
  - cloudlauch-db-rt → private, no internet route.
- Security Groups:
  - cloudlauch-app-sg: allows HTTP (80) only within VPC.
  - cloudlauch-db-sg: allows MySQL (3306) only from app subnet.

## IAM JSON Policy
'''json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "S3ListAllMyBuckets",
			"Effect": "Allow",
			"Action": "s3:ListAllMyBuckets",
			"Resource": "*"
		},
		{
			"Sid": "S3ListTwoBuckets",
			"Effect": "Allow",
			"Action": "s3:ListBucket",
			"Resource": [
				"arn:aws:s3:::cloudlauch-site-bucket",
				"arn:aws:s3:::cloudlauch-private-bucket"
			]
		},
		{
			"Sid": "S3ReadSiteObjects",
			"Effect": "Allow",
			"Action": [
				"s3:GetObject"
			],
			"Resource": "arn:aws:s3:::cloudlauch-site-bucket/*"
		},
		{
			"Sid": "S3ReadWritePrivateObjectsNoDelete",
			"Effect": "Allow",
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": "arn:aws:s3:::cloudlauch-private-bucket/*"
		},
		{
			"Sid": "S3DenyDeletesEverywhere",
			"Effect": "Deny",
			"Action": [
				"s3:DeleteObject",
				"s3:DeleteObjectVersion"
			],
			"Resource": [
				"arn:aws:s3:::cloudlauch-site-bucket/*",
				"arn:aws:s3:::cloudlauch-private-bucket/*",
				"arn:aws:s3:::cloudlauch-visible-only-bucket/*"
			]
		},
		{
			"Sid": "VPCReadOnly",
			"Effect": "Allow",
			"Action": [
				"ec2:DescribeVpcs",
				"ec2:DescribeSubnets",
				"ec2:DescribeRouteTables",
				"ec2:DescribeInternetGateways",
				"ec2:DescribeSecurityGroups",
				"ec2:DescribeAvailabilityZones"
			],
			"Resource": "*"
		}
	]
}
