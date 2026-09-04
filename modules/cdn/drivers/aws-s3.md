# AWS S3

## Installing

Install using composer:

```bash
composer require nails/driver-cdn-awslocal
```

## Configuring

### Access Credentials

The driver will need access credentials with the ability to read and write to your S3 buckets. These can be generated using AWS' IAM portal. Provided your access user has read/write permissions you can configure this however is best for your organisation, however, feel free to follow the following guidance:

{% hint style="info" %}
We recommend creating distinct groups and users per environment.
{% endhint %}

#### Create a new IAM Policy

Create an IAM Policy which will grant access to the bucket(s) you wish to use for data storage.

{% hint style="warning" %}
Remember to update your bucket name in the belo policy!
{% endhint %}

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:ListBucket",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name",
                "arn:aws:s3:::your-bucket-name/*"
            ]
        }
    ]
}
```

#### Create a new IAM user

1. Give the user a name, e.g. `CDN-PRODUCTION`
2. Grant the user `Programmatic access`
3. Attach the existing policy you created above to the user
4. Create the user
5. _**Note the Access Key ID and the Secret Access Key**_
6. [Set these details](aws-s3.md#update-driver-settings) as the driver's access credentials in Admin

### Buckets

You must configure the driver to use a bucket for each environment your application is expected to run in (i.e `PRODUCTION`, `STAGING` or `DEVELOPMENT`). We recommend using a different bucket per environment.

Once your bucket(s) are created, configure the driver using a JSON object which has the following structure:

```javascript
{
  "ENVIRONMENT": "bucket-region:bucket-name"
}
```

So, for example, a bucket in the `eu-west-2` region named `mywebsite-cdn-prod` which should be used in `PRODUCTION` would be configured like this, along side similarly named buckets for other environments:

```javascript
{
  "PRODUCTION": "eu-west-2:mywebsite-cdn-prod",
  "STAGING": "eu-west-2:mywebsite-cdn-stage",
  "DEVELOPMENT": "eu-west-2:mywebsite-cdn-dev"
}
```

Set this JSON object in [driver settings](aws-s3.md#update-driver-settings).

### **Update driver settings**

Set the details generated above in Admin.

1. Navigate to `Settings › CDN`
2. Navigate to the `Drivers` tab
3. Click `Configure` beside the `AWS` driver&#x20;
   1. _Now is a good time to double check that the driver is enabled_
4. Paste your access credentials under the `Credentials` tab
5. Click `Save Changes`
