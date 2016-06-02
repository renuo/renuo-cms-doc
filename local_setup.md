# Local Setup

For developing you want to set up the renuo-cms and the renuo-upload locally. They must be set up once and can then be used by every application that uses one or both of those services.

## Step by Step Manual

### Install renuo-cms-api

Clone git repositiory and run bin/setup:

```
git clone git@github.com:renuo/renuo-cms-api.git
cd renuo-cms-api
bin/setup
```

If you already cloned the project, you can just pull the newest version and migrate the database.

### Install renuo-upload signing

Clone git repository and run bin/setup:

```
git clone git@github.com:renuo/renuo-upload-signing.git
cd renuo-upload-signing
bin/setup
```

If you already cloned the project, you can just pull the newest version and migrate the database.

### Set Up a Amazon S3 bucket

1.  Sign in to amazon

1.  Navigate to S3 by clicking on the S3 icon:

   ![S3-Icon](s3-icon.png)

1.  Create new bucket by clicking on blue button on the top with text "create bucket":

    ![create bucket](create-bucket-button.png)

1.  Name it however you like. We request to do something like ```renuo-upload-<yourname>-development``` and choose to place where your server should be. We usually use Frankfurt.

      ![configure bucket](new-s3-bucket.png)

1.  Click on your project and navigate to Properties (right side of navigation). ![](properties-button.png) Click on the Permission section for folding it out and click on the butten called "Add CORS Configuration".

  Insert this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>*</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```
1.  Now click save, then close and after that the blue save button.

### Set UP Amazon IAM

log in to amazon if you aren't already cause you just set up your s3 bucket

navigate to IAM -> click on Identity & Access Management

navigate to Users in the navigation on the left side

click on the blue button with white writing in the form of "create user"

enter your name into the first field

press "create" which you'll find in the lower right corner its blue with white writing

click on "Show User Security Credentials" and you see your needed keys yeyy!! now you learn them by heart, as you'll need them afterwards and because its a good training for your brain. you should do stuff like that several times a day to keep your brain fit.
No better download it and store it on a place which is secure.

now click on your newly created IAM User and navigate to "Permissions". There you open the "Inline Policies" by clicking on the arrow beside it (if its not already opened)

now click on the "click here" link in the "Inline Policies" section. 

choose costum policy and click select

Set the name of your Policy to something like: ```<yourname>-s3-policy```. 

Insert this text to "Policy Document:
```json
{
  "Statement": [
    {
      "Action": [
        "s3:ListAllMyBuckets"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<your-s3-bucket-name>"
    },
    {
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<your-s3-bucket-name>/*"
    }
  ]
}
```

click on "Apply Policy"

### Configure renuo-upload-signing

Now you have to use the newly created bucket in your renuo-upload-signing. the S3_BUCKET_NAME needs the name you gave to your bucket when you set up you amazon S3 bucket. the CDN_HOST differs if you changed the location for example. the part behind the "/" will always be your bucket-name though. The S3_PUBLIC_KEY needs the "Access Key ID" which is the one you learned by heart before. Else you'll find it in the IAM of your amazon account. the same is with the S3_SECRET_KEY which is the "Secret Access Key".

```rb
S3_BUCKET_NAME: 'renuo-upload-<yourname>-development'
S3_PUBLIC_KEY: 'your public key found in Amazon IAM'
S3_SECRET_KEY: 'your secret key found in Amazon IAM'
CDN_HOST: 's3.eu-central-1.amazonaws.com/renuo-upload-<yourname>-development' #without https://, just the domain
```

### Set Up renuo-cms-demo as example project

clone it to your local machine

```
git clone git@github.com:renuo/renuo-cms-demo.git
```

set up renuo-upload for your project
```rb
API_KEYS: {"key":"h8934hghd389g89fh98h","app_name":"renuo-cms-demo","env": "development"}
```

set up renuo cms to manage content blocks for the page

```rb
CredentialPair.create(
 private_api_key: "47DTrw46jNDtt53g56Hg5MMt5",
 api_key: "T3i1A247Rd1",
 project_name: "renuo-cms-demo",
 renuo_upload_api_key: "h8934hghd389g89fh98h",
 renuo_upload_signing_url: "http://renuo-upload-signing.dev:3003/generate_policy"
)
```

navigate to the index.html of renuo-cms-demo and make a search/replace with:

search https://renuo-cms-api-demo.herokuapp.com/ replace with: //renuo-cms-api.dev:3002

### Run All Applications

run renuo-upload-signing
```
bin/run
```

run renuo-cms-api
```
bin/run
```

open index.html of renuo-cms-demo in your browser and hope that it works.