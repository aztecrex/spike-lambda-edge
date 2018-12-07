# Web Redirection @Edge Proof-of-Concept

We can easily use AWS Lambda @Edge for arbitrary web redirection logic.


## Cloudformation Resources

### Lambda Function

The Lambda function simply extracts the request object from the input event,
modifies its location URI and responds with the updated request object. In
a production application, we expect to map incoming URI locations to different
redirect destinations.

### Lambda Function Version

For @Edge, we need a specific version, not just the `$Latest`.

### DNS - Certificate Validation

For this experiment, a certificate was requested through the CLI. The
validation tokens are used in the CF template to create an appropriate
CNAME RR for validation

### DNS - Domain

The domain is the one we will redirect from. In production, we expect to
have a number of these.

### CloudFront

A Lambda @Edge function must be associated with a CloudFront distribution
to be triggered.

## Try It

### Route 53 Zone

First, be sure you have a Route 53 managed DNS zone in your account. Decide
on an incoming domain in that zone. e.g. if your DNS zone is *example.com*,
the incoming domain might be *oldhost.example.com*.  Decide on the URL
you want to redirect to. e.g. *https://www.newname.com* .

### Request a Certificate and Note the Validation Details

Request a certificate using the CLI:

`aws acm request-certificate --domain-name oldhost.example.com --validation-method DNS`.

You'll get back an ARN for the request.

Get the certificate details:

`aws acm describe-certificate  --certificate-arn arn:aws:acm:us-east-1:11111111111:certificate/your-actual-arn-value`.

This will give you the details of the DNS record needed to validate your domain in the `ResourceRecord` object.

### Create the Stack

Create the sample cloudformation stack:

```sh
aws cloudformation update-stack \
    --stack-name redirect-at-edge \
    --template-body file://cf-edgy.yaml \
    --capabilities CAPABILITY_IAM \
    --parameters \
        ParameterKey=Zone,ParameterValue=example.com \
        ParameterKey=Domain,ParameterValue=oldhost.example.com \
        ParameterKey=CertificateARN,ParameterValue=arn:aws:acm:us-east-1:11111111111:certificate/your-actual-arn-value \
        ParameterKey=CertificateValidationDomain,ParameterValue=capturedinfofromdescribecert.destination.o.cjpowered.com \
        ParameterKey=CertificateValidationValue,ParameterValue=othercapturedinfofromdescribecert.acm-validations.aws
```
