# Deployment Guide

There are three components that need deployment (links to corresponding GitHub repo):

- [aiVLE Web](https://github.com/edu-ai/aivle-web)
- [aiVLE FE](https://github.com/le0tan/aivle-fe)
- [aiVLE Worker](https://github.com/edu-ai/aivle-worker)

## Deploying Frontend

We have setup a Vercel CI in the GitHub repository - every update to the `master` branch will trigger an automatic
deployment to the main domain, and every update to the pull request will trigger a preview deployment on a temporary
domain.

## Deploying Backend

### Prepare Message Queue Broker

The message queue broker we use is [RabbitMQ](https://www.rabbitmq.com/). It is an essential dependency of Celery,
the Python task queue library we use to distribute evaluation jobs among worker nodes. Therefore you need to have
an accessible RabbitMQ instance ready before deploying aiVLE Web.

If you want to deploy RabbitMQ yourself, please refer to the [installation guide](https://www.rabbitmq.com/download.html).

If you want to use RabbitMQ as-a-service, you may take a look at [CloudAQMP](https://www.cloudamqp.com/)
or [Heroku](https://elements.heroku.com/addons/cloudamqp).

Either way you should get an address in the form `amqp://{user}:{pass}@{host}:{port}/{vhost}`, which we will refer to
as `BROKER_URI` in the future.

!!!note
    For the specification of AMQP URI, refer to [RabbitMQ URI Specification](https://www.rabbitmq.com/uri-spec.html).

!!!warning
    It is possible for the default port 5671/5672 to be blocked in your network. In this case, you may consider
    deploying the RabbitMQ server in your local network. Or try changing the default port of RabbitMQ according to
    [this guide](https://www.rabbitmq.com/configure.html#config-file).

### Deploy aiVLE Web

You may follow the setup instruction on [aiVLE Web Readme](https://github.com/edu-ai/aivle-web#readme).

There are a few extra things worth noting:

1. Please generate your Django secret key randomly using tools like [this](https://djecrety.ir/) and **don't** use
common keys like `secret_key`.
2. `FRONTEND_URL` should **not** have the trailing `/`. A valid example: `https://aivle.leotan.cn`
3. `EMAIL_HOST_PASSWORD` is your app password if your Google account has 2-factor authentication enabled (assuming
you are using Gmail as the email provider). Use [this link](https://myaccount.google.com/apppasswords) to create
an app password for aiVLE Web.
4. `DEFAULT_FROM_EMAIL` will be the default `sender` field of your verification emails. For example, 
`aiVLE No Reply <aivle.noreply@gmail.com>`.

After the deployment, you should be able to access the Django REST Framework page at the root domain (Django will
automatically redirect you to `domain.com/api/v1`). The Django admin panel is accessible at `domain.com/admin`.

## Deploy aiVLE Worker

aiVLE Worker is published as a [PyPI package](https://test.pypi.org/project/aivle-worker/). Consequently, you may follow
the [Getting Started section](https://github.com/edu-ai/aivle-worker#getting-started) of aiVLE Worker for the very easy
setup process.

You will see prompts like `worker is ready` once the client finished negotiation with the message queue system.

### What is `ACCESS_TOKEN`?

The aiVLE Web backend uses token-based authentication. In other words, in the API requests, we **never** attach the
username and password along with the request. Instead, we provide an **access token** that can be invalidated easily
without changing password for better security.

!!!note
    You may also use [aiVLE CLI](https://github.com/edu-ai/aivle-cli/releases) to get the access token.

To obtain an **access token** for your account, use `POST /dj-rest-auth/login/` API with the following form fields:

1. `username`
2. `password`

### How to Test Connectivity with the Message Queue Broker?

You may use the following code snippet (`pip install pika` first):

```python
import pika

creds = pika.PlainCredentials(username='guest', password='guest')
params = pika.ConnectionParameters(host='localhost', credentials=creds)
connection = pika.BlockingConnection(params)
channel = connection.channel()
```

You should receive a response **from the broker server** if the connection went through - it does **not** have to be
a successful response!

