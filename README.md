# django-serverless

## Tools used:
* Django
* Django-storages
* Boto3 - programatically access S3 bucket in Django application
* ECS
* Fargate
* AWS CLI - programatically upload files
* psycopg2 - managing postgres database
* postgres - replacing default sqlite database
* route53 - custom domain name

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

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/dd3b10d2-3b28-414e-a799-f29bdb67fc42)

Here we can see my static files are coming from AWS

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/bd85e9e0-90bd-467f-bedf-2195c88d196b)

I manually added these files, but it can be done programmatically. 

Admin page:
----![image](https://github.com/mfkimbell/django-serverless/assets/107063397/2d415601-02d9-4aee-833f-a53cb2b51007)

django utlizies sqlite database by default.  
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
creating admin, then logging into admin on webapp:

![image](https://github.com/mfkimbell/django-serverless/assets/107063397/d8e6f5d5-3ff4-42b2-bbd5-7aabdc6901c7)


bought domain name `mitchell-django.net`


