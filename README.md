# Cloud Formation Recipes

A collection of useful AWS Cloud Formation recipes.

If an example uses the region `us-east-1` this will usually be because the stack will only work in that region. CloudFront for example can only find ACM certificates in this region.

## S3 apex/bare domain to www domain redirect

A simple HTTP/HTTPS apex domain redirector, redirects all requests for `domain.org` to `www.domain.org`. Requires the hosted zone name to be registered in Route 53.

### Example

```sh
$ export DOMAIN=example.com
$ aws cloudformation create-stack \                                      
  --stack-name apex-redirect-${DOMAIN//\./-} \
  --template-body "$(cat ./apex-redirect.yml)" \
  --parameters \
    ParameterKey=DomainName,ParameterValue=${DOMAIN} \
  --region us-east-1
```

## S3 HTTPS website

Creates an S3 website with a CloudFront distribution in front of it. This example doesn't require the hosted zone to be in Route 53.

### Example

```sh
$ export DOMAIN=example.com
$ export ALIASES=www.example.org\,www.example.com
$ aws cloudformation create-stack \                                      
  --stack-name s3-https-website-${DOMAIN//\./-} \
  --template-body "$(cat ./s3-https-website.yml)" \
  --parameters \
    ParameterKey=DomainName,ParameterValue=${DOMAIN} \
    ParameterKey=Aliases,ParameterValue=${ALIASES} \
  --region us-east-1
```

## [Cross account] VPC Peering

Creates the peering role in the peer account and the `PeeringConnection` in the accepter account. These don't have to be cross account and will work just as well across different VPCs.

### Example

```sh
# Add assume role to peer account
$ aws cloudformation create-stack \
  --stack-name migration-assumable-role \
  --capabilities CAPABILITY_IAM \
  --template-body file://peering/cross-account-access-role.yml \
  --parameters \
    ParameterKey=PeerRequesterAccountId,ParameterValue=666 \
  --profile peer

# Add peering connection
$ aws cloudformation create-stack \
  --stack-name peer-connection \
  --template-body file://peering/peer-connection.yml \
  --parameters \
    ParameterKey=PeerRoleArn,ParameterValue=arn:aws:iam::666:role/peering-assumable-role-PeerRole-1E28H26WGHOQ1 \
    ParameterKey=VpcId,ParameterValue=vcp-666 \
    ParameterKey=PeerVpcId,ParameterValue=vcp-123 \
  --profile accepter
```

### Troubleshooting

See the [AWS docs](https://aws.amazon.com/premiumsupport/knowledge-center/cloudformation-vpc-peering-error/).