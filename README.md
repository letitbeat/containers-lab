# Docker Containers Laboratory

The purpose of this laboratory is for you to start getting familiar with the terminology and some Docker commands used to build and run an application.

The lab is composed of two parts, in the first part you will learn how to run an application from pre-built images, and in the second part you will build your first images and link them
to create a distributed service.

So let's start!

## Prerequisites

First, you will need to install Docker for your specific OS, please refer to [Docker installation](https://docs.docker.com/install/) for instructions on how to install it.

## Part 1

As previously mentioned there already available thousands of pre-built images so the goal of this part is to have an up a running [Wordpress](https://wordpress.org) website, if you are not familiar with [Wordpress](https://wordpress.org) it is a free and open source blogging tool and a content management system (CMS) based on PHP and MySQL used by millions of websites.

You will have to pull and run both images [Wordpress](https://hub.docker.com/_/wordpress/) and [MySQL](https://hub.docker.com/_/mysql/) in order to achieve the goal of this part. For instructions on how to run each of these images please refer to the Docker store documentation.

> Please make sure you pull the version 5.7 of MySQL

## Part 2

### 2.1

This will be a Python Flask application that returns a random "Hello, world!" message every time is requested.

Start by creating a directory called `api` that will contain the following files:

* app.py
* requirements.txt
* Dockerfile

#### app.py

Create the app.py with the following content:

```python
from flask import Flask, jsonify
from flask_cors import CORS
import random, json

app = Flask(__name__)
CORS(app)

class Greeting:
    def __init__(self, msg, language):
        self.msg = msg
        self.language = language

greetings = [
    Greeting("Hello, World!", "English"),
    Greeting("Bonjour le monde!", "French"),
    Greeting("Hola, mundo!", "Spanish"),
    Greeting("Hallo Welt!", "German"),
    Greeting("Прывітанне Сусвет!", "Belarussian"),
    Greeting("你好世界！", "Chinese" ),
    Greeting("Ciao mondo!", "Italian" ),
    Greeting("Hej världen!", "Swedish"),
    Greeting("Chào thế giới!", "Vietnamese"),
    Greeting("Ahoj světe!", "Czech")
]

@app.route('/')
def index():
    greeting = random.choice(greetings)
    return json.dumps(greeting.__dict__, ensure_ascii=False)

if __name__ == "__main__":
    app.run(host="0.0.0.0")
```

#### requirements.txt

In order to install the Python modules required for our app, we need to create a file called requirements.txt and add the following lines to that file:

```
Flask==1.0.2
flask-cors==3.0.7
```

#### Dockerfile

We want to create a Docker image with this web app. As mentioned above, all user images are based on a base image. Since our application is written in Python, we will build our own Python image based on Alpine. We'll do that using a Dockerfile.

A Dockerfile is a text file that contains a list of commands that the Docker daemon calls while creating an image. The Dockerfile contains all the information that Docker needs to know to run the app — a base Docker image to run from, location of your project code, any dependencies it has, and what commands to run at start-up. It is a simple way to automate the image creation process. The best part is that the commands you write in a Dockerfile are almost identical to their equivalent Linux commands. This means you don't really have to learn new syntax to create your own Dockerfiles.

1. Create a file called Dockerfile, and add content to it as described below.
We'll start by specifying our base image, using the `FROM` keyword:

```
FROM alpine:3.9
```

2. The next step usually is to write the commands of copying the files and installing the dependencies. But first we will install the Python pip package to the alpine linux distribution. This will not just install the pip package but any other dependencies too, which includes the python interpreter. Add the following RUN command next:

```
RUN apk add --update python3 && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools
```
3. Let's add the files that make up the Flask Application.

Install all Python requirements for our app to run. This will be accomplished by adding the lines:
```
COPY requirements.txt /usr/src/app/
RUN pip3 install --no-cache-dir -r /usr/src/app/requirements.txt
```

Copy the files you have created earlier into our image by using COPY command.

```
COPY app.py /usr/src/app/
```
4. Specify the port number which needs to be exposed. Since our flask app is running on 5000 that's what we'll expose.

```
EXPOSE 5000
```

5. The last step is the command for running the application which is simply - `python ./app.py`. Use the CMD command to do that:

```
CMD ["python", "/usr/src/app/app.py"]
```

#### Build the image

Now that you have your Dockerfile, you can build your image. The docker build command does the heavy-lifting of creating a docker image from a Dockerfile.

The `docker build` command is quite simple - it takes an optional tag name with the `-t` flag, and the location of the directory containing the `Dockerfile` - the . indicates the current directory:

```
$ docker build -t lab-api .
Sending build context to Docker daemon   5.12kB
Step 1/7 : FROM alpine:3.9
 ---> 5cb3aa00f899
Step 2/7 : RUN apk add --update python3 &&     python3 -m ensurepip &&     rm -r /usr/lib/python*/ensurepip &&     pip3 install --upgrade pip setuptools
 ---> Using cache
 ---> 134ff5ad673e
Step 3/7 : COPY requirements.txt /usr/src/app/
 ---> 7db8a3c9ca64
Step 4/7 : RUN pip3 install --no-cache-dir -r /usr/src/app/requirements.txt
 ---> Running in 26f50c33c40d
Collecting Flask==1.0.2 (from -r /usr/src/app/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
Collecting flask-cors==3.0.7 (from -r /usr/src/app/requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/65/cb/683f71ff8daa3aea0a5cbb276074de39f9ab66d3fbb8ad5efb5bb83e90d2/Flask_Cors-3.0.7-py2.py3-none-any.whl
Collecting Jinja2>=2.10 (from Flask==1.0.2->-r /usr/src/app/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
Collecting click>=5.1 (from Flask==1.0.2->-r /usr/src/app/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting itsdangerous>=0.24 (from Flask==1.0.2->-r /usr/src/app/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting Werkzeug>=0.14 (from Flask==1.0.2->-r /usr/src/app/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/29/5e/d54398f8ee78166d2cf07e46d19096e55aba506e44de998a1ad85b83ec8d/Werkzeug-0.15.0-py2.py3-none-any.whl (328kB)
Collecting Six (from flask-cors==3.0.7->-r /usr/src/app/requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10->Flask==1.0.2->-r /usr/src/app/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/b9/2e/64db92e53b86efccfaea71321f597fa2e1b2bd3853d8ce658568f7a13094/MarkupSafe-1.1.1.tar.gz
Installing collected packages: MarkupSafe, Jinja2, click, itsdangerous, Werkzeug, Flask, Six, flask-cors
  Running setup.py install for MarkupSafe: started
    Running setup.py install for MarkupSafe: finished with status 'done'
Successfully installed Flask-1.0.2 Jinja2-2.10 MarkupSafe-1.1.1 Six-1.12.0 Werkzeug-0.15.0 click-7.0 flask-cors-3.0.7 itsdangerous-1.1.0
Removing intermediate container 26f50c33c40d
 ---> 304a718dbe05
Step 5/7 : COPY app.py /usr/src/app/
 ---> 6fd31a586b1f
Step 6/7 : EXPOSE 5000
 ---> Running in 4a98b1632f0f
Removing intermediate container 4a98b1632f0f
 ---> 2c35dcd0cb7e
Step 7/7 : CMD ["python3", "/usr/src/app/app.py"]
 ---> Running in a179dc33566e
Removing intermediate container a179dc33566e
 ---> d89454bcb4da
Successfully built d89454bcb4da
```

If everything went well, your image should be ready! Run `docker images` and see if your image `lab-api` shows.

#### Run the image

Now we can run the image and see if actually works:

```
$ docker run -p 8888:5000 --name lab-api lab-api
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Head over to `http://localhost:8888` and your app should be live.

You see and output similar to this:
```
{"msg": "Hola, mundo!", "language": "Spanish"}
```

Hit the Refresh button in the web browser to see a few more messages.

### 2.2

Now that we have the API, it's time to create the front-end that will be requesting the messages and displaying in a proper way.

Start by creating a directory called `web` that will contain the following files:

* index.html
* app.js
* Dockerfile

#### index.html

Create an index.html with the following content:

```html
<html>
    <head>
        <title>Amazing Hello World Service</title>
        <style>
            body {
            font-family: 'Lucida Console', 'Courier New', monospace;
            font-size: 26pt;
            line-height: 1.2em;
            }
            .content {
            width: 760px;
            text-align: center;
            margin: 1em auto;
            }
        </style>
    </head>
    <body>
        <div>
            <h3>My Amazing Hello World Service!</h3>
            <div class="content">
                <div id="msg">Hello World!</div>
                <p id="lang">English</p>
            </div>
        </div>
        <script src="app.js"></script>
    </body>
</html>
```

#### app.js

Also create the `app.js` with the following content inside:
```javascript
fetch('http://localhost:8888/')
.then(function (response) {
    return response.json();
})
.then(function (json) {
    document.querySelector("#msg").innerHTML = json.msg;
    document.querySelector("#lang").innerHTML = '<i>-' + json.language + '</i>';
})
.catch(error => {
    console.log("Fetching trees data failed", error);
});
```

#### Dockerfile

Similarly to the `api` Docker file, let's create the image for the web application, but this time a bit more simpler

1. We will be using the NGINX to serve our static content.

```
FROM nginx:latest
```

2. Now, we `COPY` the files created in the previous step

```
COPY index.html /usr/share/nginx/html
COPY app.js /usr/share/nginx/html
```

That's all!

#### Build the web app image

```
$ docker build -t lab-web .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM nginx
 ---> 881bd08c0b08
Step 2/2 : COPY app /usr/share/nginx/html
 ---> 2a3f3e03c54a
Successfully built 2a3f3e03c54a
Successfully tagged lab-web:latest
```
#### Run the image

```
$ docker run -p 8080:80 lab-web lab-web
```

And now just go ahead to `http://localhost:8080/` to see your new 'Amazing Hello World Service' up and running!

** Optional: You can go further and use [Docker compose](https://docs.docker.com/compose/) to build and deploy this web application.

>The second part of this laboratory is partially based on [Web apps with Docker](https://github.com/docker/labs/blob/master/beginner/chapters/webapps.md) lab.




