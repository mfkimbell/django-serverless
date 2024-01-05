# django-serverless

## Tools used:
* Django
* Django-storages
* Boto3 - programatically access S3 bucket in Django application
* ECS
* Fargate
* AWS CLI - programatically upload files

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


