# django-serverless

## Tools used:

* AWS CLI - programatically upload files
* Boto3 - programatically access S3 bucket in Django application
* Certificate Manager - SSL certificate for using 'https'
* Django -
* Django-environ - setting up enviornment variables for secret values
* Django-storages -
* Docker - containerization of application
* ECR - container registry for storing application image
* ECS - container management
* Fargate - serverless launch type for running docker container
* Guinicorn - wsgi http server used to build connection between django application and AWS
* Psycopg2 - managing postgres database
* Postgres - replacing default sqlite database
* Route53 - custom domain names for webapp


## To Run Locally:

(for me personally, I had to run `Set-ExecutionPolicy -ExecutionPolicy Unrestricted` to execute the active script on Windows11, or you can open powershell as admin)

```
cd django/simply/simply
virtualenv venv
pip install -r requirements.txt
cd venv/Scripts
activate (and simply write `deactivate` to close down the virtualenv)
python manage.py runserver
```

And you can write `deactivate` to shut down the virtual environment

## Webapp demo:

![django display](https://github.com/mfkimbell/django-serverless/assets/107063397/264ce417-bd72-4389-9080-4fa64f880817)

## AWS Implementation:

-created user group "developer" with AdminAccess, IAMChangePasswordAccess, AmazonEC2ContainerRegistryFullAccess, and AmazonECS_FullAccess as the permission policies.
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/b3c797c5-6960-42dd-9475-eca64f863bff)

-created user "mitchell" with access to that role

-created account alias "mitchell-django"

-new sign in link "https://mitchell-django.signin.aws.amazon.com/console"

-create access key for IAM user

-install aws cli

I create a bucket policy so that anyone can call getObject in my S3 bucket:

``` json
{
  "Id": "Policy1704489125788",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1704489124269",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::django-static-151/*",
      "Principal": "*"
    }
  ]
}
```

-in the settings.py of my django application, I add access keys and custom domain

```
AWS_ACCESS_KEY_ID = "AKIAW3MEFFIRVJ2I7SLG"
AWS_SECRET_ACCESS_KEY = "****************************"
AWS_STORAGE_BUCKET_NAME = 'django-static-151'
STORAGES = {"staticfiles": {"BACKEND": "storages.backends.s3boto3.S3StaticStorage"}}
AWS_S3_CUSTOM_DOMAIN = '%s.s3.amazonaws.com' % AWS_STORAGE_BUCKET_NAME
```

I manually added these files, but it can be done programmatically: 

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/bd85e9e0-90bd-467f-bedf-2195c88d196b)

Here we can see my static files are coming from AWS:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/dd3b10d2-3b28-414e-a799-f29bdb67fc42)



Django utlizies sqlite database by default, so I change it to PostgreSQL.

``` python
DATABASES = {

    'default': {

        'ENGINE': 'django.db.backends.postgresql',

        'NAME': 'demo_db', 

        'USER': 'mfkimbell',

        'PASSWORD': 'password',

        'HOST' : 'database-1.cpkm2gqmy8u4.us-east-2.rds.amazonaws.com',

        'PORT': '5432',

    }

}
```
Creating admin, then logging into admin on webapp. I personally never utilized these features, but it can be used to manage data in the django application:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/d8e6f5d5-3ff4-42b2-bbd5-7aabdc6901c7)


bought domain name `mitchell-django.net`
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/e90c00ef-34c4-446c-9cdf-6e950810445a)
Registered the domains in Certificate Manager
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/aa6db38a-ea1f-47d4-abaa-af588f880eb6)
Added CNAME records to Route53
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/4cf37b38-edf4-49f6-b77b-e2d59413736d)


and we add the domain to our settings.py

```python
ALLOWED_HOSTS = ['www.mitchell-django.com','mitchell-django.com','*']
CSRF_TRUSTED_ORIGINS = ['https://www.mitchell-django.com', 'https://mitchell-django.com']
```

Now we create the Dockerfile:
-I add PYTHONUNBUFFERED=1 since it will send python output to our container logs
-I also specify a port number so it can be accessed outside the container
```python
FROM --platform=linux/amd64 python:3.11-bullseye

ENV PYTHONUNBUFFERED=1

WORKDIR /simply

COPY requirements.txt .

RUN pip3 install -r requirements.txt

COPY . .

CMD python manage.py runserver 0.0.0.0:8000
```

I then run
```
docker build -t simply2 .
docker run -p 8888:8000 simply2
```

Here we can see the application successfully running on the container:
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/fe584e64-25b3-4f50-9aae-efbe7cd32861)

Next I upload my container to ECR with the following commands:

```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 471112952355.dkr.ecr.us-east-2.amazonaws.com
docker tag simply2:latest 471112952355.dkr.ecr.us-east-1.amazonaws.com/my-first-repository:latest
docker push 471112952355.dkr.ecr.us-east-1.amazonaws.com/my-first-repository:latest
```

Fargate uses dynamic IPs, that's why we use '*'. 

-ECS tasks come from "task definitions", which are json templates that tell it what to do


I use an application load balancer because this applicaiton uses https.

I make a Security Group `DemoAppLB-SG`, which I attack to a Load Balancer `DemoAppLB`. I add to the Load Balancer's listener Target Group `DemoAppTG` and I direct to IP since I plan to use Fargate. I specify the Target Group to direct to HTTP port 8000 so our Load Balancer can communicate to the docker container (which if you remember, runs on port 8000). 

Load Balancer --> Listener --> Target Group --> Application
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/9ca34762-7f2b-4ed0-a02f-e83478f89401)

Currently, I have it set up so that anyone trying to connect via HTTP will be automatically redirected to HTTPS:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/d8b5c9ac-1e5e-4203-996d-bf964a50fb0b)

We can see the target group on port 8000:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/b018055d-d96b-464b-b0d7-174a0381642b)

And we can see all of the Route53 routing:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/def449a3-a732-4e6b-b053-93ec727e165c)

The Type A records point the the Load Balancer, and our CNAME records are used for SSL certificate validation.

Now I need to create a Task Definition. I create a container named `DemoAppContainer` and link it to the Container URI that I uploaded to ECR.

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/2abe5779-caea-4d59-af89-87002adab17d)



I also create an ECS cluster `DemoAppCluster`. Each instance in a cluster is called a node. Clusters can contain a mix of tasks that are hosted on AWS Fargate, Amazon EC2 instances, or external instances. You can monitor the creation of the cluster in **CloudFormation**. 

A **Service** is used to guarantee that you always have some number of Tasks running at all times. If a Task's container exits due to an error, or the underlying EC2 instance fails and is replaced, the ECS Service will replace the failed Task. This is why we create Clusters so that the Service has plenty of resources in terms of CPU, Memory and Network ports to use. To us it doesn't really matter which instance Tasks run on so long as they run. A Service configuration references a Task definition. A Service is responsible for creating Tasks. I create a service called `DemoAppService` which will use the Load Balancer and Target groups previously created. 

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/8b0bc3c8-143d-48ad-a1d9-f6288c5a9272)

We can see the launch type is specifically Fargate:
![image](https://github.com/mfkimbell/django-serverless/assets/107063397/14f1803f-1587-4562-bfdf-68705dddcbda)


And here we can see my application running on `www.mitchell-django.net`:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/6c1d090d-a27e-4a15-8ae0-5719bb5b93ca)

A
