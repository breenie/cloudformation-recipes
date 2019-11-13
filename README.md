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