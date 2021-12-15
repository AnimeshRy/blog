---
layout: post
title: Running Cron Jobs in Django
subtitle: Bye Celery!
categories: Django
tags: [unix, web-applications, django, python]
---

If you are our UNIX or Linux user, you must have heard of cron. It's one of the most useful tools in Unix systems although usually used in sysadmin jobs but still, you might need one too !?

### **According to Wikipedia:**

> *The software utility **cron** is also known as **cron job** is a time-based job scheduler in Unix-like computer operating systems. Users who set up and maintain software environments use cron to schedule jobs (commands or shell scripts) to run periodically at fixed times, dates, or intervals.*
>

Today we'll talk about setting up cron jobs in Django.

Firstly it depends on the specific use case if you even need a cron job. You might need a distributed task queue like Celery for your case.

If you are confused among the two, check out this [StackOverflow](https://stackoverflow.com/questions/16232572/distributed-task-queues-ex-celery-vs-crontab-scripts#:~:text=What%20celery%20brings%20to%20the,a%20minimum%20of%20one%20minute.) answer.

### Let's Begin

We need the package [django-crontab](https://pypi.org/project/django-crontab/) for running cron jobs.

### Setup

- Install the package and add it as a app inside installed apps

```bash
pip install django-crontab
```

- Add the app to installed apps. I also have a `test_app` app inside my project for demonstration. Also, add the `CRONJOBS` setting inside settings.py

```python
CRONJOBS = [
    ('*/5 * * * *', 'test_app.cron.my_scheduled_job')
]
# /5 represent every 5 minutes
# my_scheduled_job is the jobs we'll add
```

```python
INSTALLED_APPS = (
    'django_crontab',
		'test_app',
    ...
)
```

- Create a `[cron.py](http://cron.py)` inside the test_app and add the following -

```python
def my_scheduled_job():
  pass
```

- finally run this command to add all defined jobs from CRONJOBS to crontab (of the user which you are running this command with):

```bash
python manage.py crontab add
```

- show current active jobs of this project:

```bash
python manage.py crontab show
```

- removing all defined jobs is straight forward:

```bash
python manage.py crontab remove
```

### Example -

### Create a model object every minute (*Simple*)

```python
from .models import Test # importing the test model

def my_scheduled_job():
  Test.objects.create(name='test')
```

- You can adjust the timing in the `CRONJOBS`  array we added in `settings.py` file earlier.

### Check if a document is expired ? (*Intermediate*)

```python
# test_app.models - Add a test Document model
class Document(models.Model):
		uploaded_file = models.FileField(upload_to='images/')
		expiration_date = models.DateField()
		expired = models.BooleanField(default=False)
		updated = models.DateTimeField(auto_now=True)
		created = models.DateTimeField(auto_now_add=True)

		def __str__(self):
				return str(self.id)
```

```python
# test_app.cron - Check today's date on the file
from .models import Document
from datetime import datetime

def document_expired_check():
		today = datetime.today.strftime('%Y-%m-%d')
		qs = Document.objects.filter(expired=False)
		for doc in qs:
				exp = doc.expiration_date.strftime('%Y-%m-%d')
				if exp < today:
						doc.expired=True
						doc.save()
```

```python
# settings.py - Add the job with a check performed every minute
CRONJOBS = [
    ('*/5 * * * *', 'test_app.cron.my_scheduled_job'),
		('*/1 * * * *', 'test_app.cron.document_expired_check')
]
```

- You'll need to run the `cronjob` command above to add it.

You can try out more advanced examples if you need to but this will be all for this article.

Author - [Animesh Singh](https://www.iamanimesh.tech)
