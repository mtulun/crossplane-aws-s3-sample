# Crossplane Example S3 Bucket Creation

* Install Configuration Package
    - kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-aws:v1.8.1

* Get AWS Account Keyfile
    - WS_PROFILE=default && echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf

* Create a Provider Secret
    - kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=./creds.conf

* Configure the Provider

    ````
    apiVersion: aws.crossplane.io/v1beta1
    kind: ProviderConfig
    metadata:
        name: default
    spec:
    credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: creds

    ````
* Apply the Provider.yaml
    - kubectl apply -f <Provider.yaml>

* Configure the ProviderConfig

    ````
    apiVersion: aws.crossplane.io/v1beta1
    kind: ProviderConfig
    metadata:
        name: awsconfig
    spec:
        credentials:
            source: Secret
            secretRef:
                namespace: crossplane-system
                name: aws-secret-creds
                key: creds

    ````

* At last apply the YAML formatted s3_bucket file.
    - kubectl apply -f <s3.yaml>

### Notes

    - After a bucket is created, folder prefixes and files are not lost when an update is made on that bucket.

## For Bucket Policy we can configure IAM JSON policy elements: Condition

    - The Condition element (or Condition block) lets you specify conditions for
      when a policy is in effect. The Condition element is optional. In the Condition 
      element, you build expressions in which you use condition operators 
      (equal, less than, etc.) to match the condition keys and values in the policy against
      keys and values in the request context.

            "Condition" : { "{condition-operator}" : { "{condition-key}" : "{condition-value}" }}
    
    - The condition key that you specify can be a global condition key or a service-specific
      condition key. Global condition keys have the aws: prefix. Service-specific condition keys
      have the service's prefix. For example, Amazon EC2 lets you write a condition using the ec2:InstanceType key,
      which is unique to that service. To view service-specific IAM condition keys with the
      iam: prefix, see IAM and AWS STS condition context keys.

    - Condition key names are not case-sensitive. For example, including the aws:SourceIP condition
      key is equivalent to testing for AWS:SourceIp. Case-sensitivity of condition key values depends
      on the condition operator that you use. For example, the following condition includes the StringEquals
      operator to ensure that only requests made by johndoe match. Users named JohnDoe are denied access.


            "Condition" : { "StringEquals" : { "aws:username" : "johndoe" }}

    - The following condition uses the StringEqualsIgnoreCase operator to match users named johndoe or JohnDoe.
        

            "Condition" : { "StringEqualsIgnoreCase" : { "aws:username" : "johndoe" }}

## The condition block

* The following example shows the basic format of a ```Condition``` element:

    ````
        "Condition": {"StringLike": {"s3:prefix": ["janedoe/*"]}}
    ````
    - A value from the request is represented by a condition key, in this case s3:prefix. The context
      key value is compared to a value that you specify as a literal value, such as janedoe/*. 
      The type of comparison to make is specified by the condition operator (here, StringLike).
      You can create conditions that compare strings, dates, numbers, and more using typical Boolean
      comparisons such as equals, greater than, and less than. When you use string operators or ARN operators,
      you can also use a policy variable in the condition value.
      The following example includes the aws:username variable.

    ````
        "Condition": {"StringLike": {"s3:prefix": ["${aws:username}/*"]}}
    ````