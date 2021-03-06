# CI/CD pipeline
## Using free and hosted services

![](https://cdn-images-1.medium.com/max/800/1*z6dyw9e-wPWYxRYZWhSXKQ.png)

Software developers are problem solvers. Although they can be very tenacious about finding solutions, the moment they have one they want to share it with the world; the feeling of shipping code is great. Code that is never executed for users is no more than a digital waste product. To prevent building waste, modern software developers ship functionality to their users in short iterations and small increments.

A way to ship code in small increments and iterations is by using a Continuous Integration and Continuous Deployment, or CI/CD, pipeline. In this tutorial we’ll go through all the steps in setting up such a pipeline using free and hosted services. From start to end this tutorial shows you in 9 steps how to:

1. Write a little Python program (not Hello World)
2. Add some automated tests for the program
3. Push your code to GitHub
4. Setup Travis CI to continuously run your automated tests
5. Setup Better Code Hub to continuously check your code quality
6. Turn the Python program into a web app
7. Create a Docker image for the web app
8. Push the Docker image to Docker Hub
9. Deploy the Docker image to Heroku

The goal of this tutorial is to show you the essential building blocks of a modern software development pipeline, and how to assemble those blocks. It’s not about any programming language or software development methodology in particular. Maybe some of the building building blocks do not match your technology stack of choice. Even if that’s the case I still advice you to go through all the steps. Once you have a basic understanding of the complete end-to-end pipeline you can make it easily fit your specific needs.

Update August 2017: Eran Barlev from codefresh.io published a short video on LinkedIn demonstrating how Codefresh can replace some of the building blocks described in this tutorial. Since no two software products are the same, development pipelines can vary too. Be sure to check out Eran’s video to see an alternative implementation of a modern CI/CD pipeline.

### Step 1: Write a little buzz generator
We need a small piece of software that will travel through all phases of the pipeline, from your laptop to the cloud. In our case this piece of software is a little CI/CD buzz generator program written in Python.

Create a new directory (let’s say we call it ‘cicd-buzz’). Inside this directory create another directory called ‘buzz’ and in this directory save the snippet below to a file called ‘generator.py’.

```python
from __future__ import print_function
import random

buzz = ('continuous testing', 'continuous integration',
    'continuous deployment', 'continuous improvement', 'devops')
adjectives = ('complete', 'modern', 'self-service', 'integrated', 'end-to-end')
adverbs = ('remarkably', 'enormously', 'substantially', 'significantly',
    'seriously')
verbs = ('accelerates', 'improves', 'enhances', 'revamps', 'boosts')

def sample(l, n = 1):
    result = random.sample(l, n)
    if n == 1:
        return result[0]
    return result

def generate_buzz():
    buzz_terms = sample(buzz, 2)
    phrase = ' '.join([sample(adjectives), buzz_terms[0], sample(adverbs),
        sample(verbs), buzz_terms[1]])
    return phrase.title()

if __name__ == "__main__":
    print(generate_buzz())
```    
    
Also create a new empty file called ‘__init__.py’ in the same directory. Your project structure should look like this now:

```
cicd-buzz/
  buzz/
    __init__.py
    generator.py
```

You should be able to run this Python script from the command line inside the ‘buzz’ directory:

```
[cicd-buzz/buzz] $ python generator.py
End-To-End Devops Enormously Boosts Continuous Testing
```

Try it a couple of times, it’s fun:

```
[cicd-buzz/buzz] $ python generator.py
Complete Continuous Improvement Enormously Improves Devops
[cicd-buzz/buzz] $ python generator.py
Modern Devops Remarkably Improves Continuous Testing
```

### Step 2: Add automated tests

Continuous delivery pipelines only makes sense when you have a significant amount of automated tests that prevents you from continuously shipping broken software. To get a proper set of unit-tests for our buzz generator create a new directory called ‘tests’ in the root of your project directory and save the the snippet below to a new file in the ‘tests’ directory. Call this file ‘test_generator.py’:

```python

import unittest

from buzz import generator

def test_sample_single_word():
    l = ('foo', 'bar', 'foobar')
    word = generator.sample(l)
    assert word in l

def test_sample_multiple_words():
    l = ('foo', 'bar', 'foobar')
    words = generator.sample(l, 2)
    assert len(words) == 2
    assert words[0] in l
    assert words[1] in l
    assert words[0] is not words[1]

def test_generate_buzz_of_at_least_five_words():
    phrase = generator.generate_buzz()
    assert len(phrase.split()) >= 5
    
```
 
To run the tests we’ll be using the ‘pytest’ framework. To install pytest we’ll be using a Python Virtual Environment (‘virtualenv’). Don’t worry, that’s easier done than said. First, make sure you have virtualenv installed so you can execute the following command inside your project directory:

```
[cicd-buzz] $ virtualenv venv
```

This should create a new directory venv. To start using this environment type:
This does not work on windows
```
[cicd-buzz] $ source venv/bin/activate
(venv) [cicd-buzz] $
```
Navigate to Scripts folder in venv and run the activate bat script
```
C:\Learn\cicd-buzz\venv\Scripts>activate
```

Next, create a new file called ‘requirements.txt’ that lists the pytest dependency:

```
pytest==3.0.6
```

To download dependencies listed in the requirements file you’ll need to execute the ‘pip’ command:

```
(venv) [cicd-buzz] $ pip install -r requirements.txt
```

After you’ve completed all steps above, the root of your project directory should look like this:

```
cicd-buzz/
  buzz/
  requirements.txt
  tests/
  venv/
```

Inside the virtual environment you can now run the unit-tests in the ‘test_generator.py’ file:
```
(venv) [cicd-buzz] $ python -m pytest -v tests/test_generator.py
```
The output should look something like:
``` python
============================= test session starts =============================
platform win32 -- Python 3.7.0, pytest-3.0.6, py-1.6.0, pluggy-0.4.0 -- C:\Learn\cicd-buzz\venv\Scripts\python.exe
cachedir: .cache
rootdir: C:\Learn\cicd-buzz, inifile:
collected 3 items

tests/test_generator.py::test_sample_single_word PASSED
tests/test_generator.py::test_sample_multiple_words PASSED
tests/test_generator.py::test_generate_buzz_of_at_least_five_words PASSED

========================== 3 passed in 0.08 seconds ===========================
```

### Step 3: Put the code on GitHub

Login to GitHub (get an account first if you don’t have one already) and create a new public repository called ‘cicd-buzz’.

Inside your project directory create a new file called ‘.gitignore’ containing only a single line:
```
venv
```

This will prevent git from adding the virtualenv to our repo. Now it’s time to initialize Git locally and push your code to GitHub:
```
[cicd-buzz] $ git init
[cicd-buzz] $ git add *
[cicd-buzz] $ git commit -m "Initial commit"
[cicd-buzz] $ git remote add origin git@github.com:<YOUR_GITHUB_USERNAME>/cicd-buzz.git
[cicd-buzz] $ git push -u origin master
```
If the last command above complains about access rights, make sure you’ve added your SSH key to your GitHub account.


### Step 4: Connect Travis CI to run the tests on every commit

Travis CI is a hosted service for Continuous Integration work. It’s free for public GitHub repositories and getting a Travis CI account is just a matter of visiting https://travis-ci.org and logging in with your GitHub credentials.

Enabling Travis CI to start a build at each Push and Pull Request for your repository is as easy as flipping the switch in front of your GitHub cicd-buzz repository (click the ‘Sync account’ button in case your repository is not yet visible) 

The last step in activating Travis CI is to add a ‘.travis.yml’ file in the root of your project directory. For our buzz generator this file should contain:

```
language: python
script:
  - python -m pytest -v
 ```
Add the file to Git, then commit and Push your changes:
```
[cicd-buzz] $ git add .travis.yml
[cicd-buzz] $ git commit -m "Add Travis CI configuration"
[cicd-buzz] $ git push
```

Go to the Travis CI dashboard. After a short amount of time Travis should notice your code changes and start the build/test process. The output log shows the execution of the unit-tests: https://travis-ci.org/Develop-X/cicd-buzz

![](https://cdn-images-1.medium.com/max/800/1*VzPJaJ33Jz6l9WvrhBgu7g.png)

### Step 5: Add Better Code Hub to your pipeline
Now that we have a well oiled pipeline that continuously checks the functionality of our code with automated tests, the temptation is strong to focus on functionality and forget about quality. Better Code Hub is a hosted platform that checks the quality of your code according to the 10 guidelines for maintainable, future proof code. Better Code Hub is a watchdog that continuously monitors our development work (literally every push to GitHub) and notifies you when the quality is at risk.

Better Code Hub is, like Travis CI, a service that seamlessly integrates with GitHub. To attach it to our repo, go to https://bettercodehub.com and choose the login button that says Free.

After logging in with your GitHub credentials, the next page lists all your GitHub repositories. Find the repository called ‘cicd-buzz’ and press the play button. Better Code Hub will then ask you if it’s fine to run the analysis with the default configuration. Click ‘Go’ and wait a few seconds, the analysis report should now be on your screen.

If you want Better Code Hub to run for every Push and Pull Request on your repo (just like Travis CI), toggle the pull-request icon that is displayed on the bottom-left part of the repo card:

![](https://user-images.githubusercontent.com/8856857/46511557-a000fd80-c892-11e8-942d-b8fa95863f1c.png)

### Step 6: Turn the buzz generator into a simple web app

Nice job! You already have a continuous integration pipeline that checks for functionality and quality at this point. Next step is to continuously deploy your software whenever all tests pass.

Since we will deploy the software to Heroku as a web app, we first need to write a little Python Flask wrapper around our buzz generator to make the program respond to HTTP requests and output HTML. Add the code below in a file called ‘app.py’ in the root of your project directory:

```python
import os
import signal
from flask import Flask
from buzz import generator

app = Flask(__name__)

signal.signal(signal.SIGINT, lambda s, f: os._exit(0))

@app.route("/")
def generate_buzz():
    page = '<html><body><h1>'
    page += generator.generate_buzz()
    page += '</h1></body></html>'
    return page

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=int(os.getenv('PORT', 5000)))

```
Also add another line to your ‘requirements.txt’ file for the Flask framework:
```
pytest==3.0.6
Flask==0.12
```
And install the new dependency:
```
(venv) [cicd-buzz] $ pip install -r requirements.txt
```
You can now run the web app on your laptop:
```
[cicd-buzz] $ python app.py
```
* Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
Open the location http://localhost:5000 in a browser and admire your achievement. Hit refresh a couple of times, just for fun.

Finally, don’t forget to add your commit and push your changes:
```
[cicd-buzz] $ git add app.py
[cicd-buzz] $ git add requirements.txt
[cicd-buzz] $ git commit -m "Step 6"
[cicd-buzz] $ git push
```
And enjoy watching Travis CI and Better Code Hub picking up this push!

### Step 7: Containerize your web app with docker

We’ll use Docker to create a single self-contained, deployable unit of our web app. For a simple Python Flask app this may look like a lot of overhead but deploying different versions of your code base as a small, self-contained unit has a lot of benefits when your system grows over time.

Assuming you have Docker up and running, add the following to a new file called ‘Dockerfile’ in the root of your project directory:

``` python
FROM alpine:3.5
RUN apk add --update python py-pip
COPY requirements.txt /src/requirements.txt
RUN pip install -r /src/requirements.txt
COPY app.py /src
COPY buzz /src/buzz
CMD python /src/app.py
```

The above tells docker to pick the alpine base image, install Python and pip, and also install our web app. The last line tells docker to launch the web app whenever the container is launched.

You should be able to build an image of this Docker configuration and launch it (depending on your OS configuration you might need to put sudo in front of the commands below):

```
[cicd-buzz] $ docker build -t cicd-buzz .
[cicd-buzz] $ docker run -p 5000:5000 --rm -it cicd-buzz
```

Again, don’t forget to add your commit and push your changes:
```
[cicd-buzz] $ git add Dockerfile
[cicd-buzz] $ git commit -m "Step 7"
[cicd-buzz] $ git push
```
### Step 8: Deploy to Docker Hub

Deploying your containers to a central Docker image registry, such as Docker Hub makes it much easier to share your containers in different environments or to go back to a previous version. To complete this step you’ll need to sign up at https://docker.com and add the following to a file called ‘deploy_dockerhub.sh’ in a new directory called ‘.travis’ in your project directory:

``` python
#!/bin/sh
docker login -e $DOCKER_EMAIL -u $DOCKER_USER -p $DOCKER_PASS
if [ "$TRAVIS_BRANCH" = "master" ]; then
    TAG="latest"
else
    TAG="$TRAVIS_BRANCH"
fi
docker build -f Dockerfile -t $TRAVIS_REPO_SLUG:$TAG .
docker push $TRAVIS_REPO_SLUG
```
The script above will be called by Travis CI at the end of each pipeline build and will create a new deployable Docker image for that pipeline build. The script requires 3 environment variables that you can set under the ‘settings’ view of your cicd-buzz repo at Travis CI:

![](https://user-images.githubusercontent.com/8856857/46588845-2a916900-caee-11e8-9390-db214a0fadbb.png)

To have Travis CI deploy you Docker image to Docker Hub for each code push to your GitHub repository, modify your ‘.travis.yml’ file so that it looks like:

```
sudo: required

services:
  - docker

language: python

script:
  - python -m pytest -v

after_success:
  - sh .travis/deploy_dockerhub.sh
  
```

After committing and pushing these changes (and waiting for Travis CI to complete the full pipeline) you should be able to launch your Docker image straight from Docker Hub:
```
[cicd-buzz] $ docker run -p5000:5000 --rm -it <YOUR_DOCKER_USERNAME>/cicd-buzz:latest
```
### Step 9: Deploy to Heroku
Heroku is a cloud platform for hosting small and scalable web applications. It offers a free plan so go to https://signup.heroku.com and sign up if you haven’t already done so.

Install the Heroku command-line toolbelt and from the root of your project directory execute the following commands:

```
[cicd-buzz] $ heroku login
[cicd-buzz] $ heroku create
Creating app… done, ⬢ fathomless-inlet-53225
https://fathomless-inlet-53225.herokuapp.com/ | https://git.heroku.com/fathomless-inlet-53225.git
[cicd-buzz] $ heroku plugins:install heroku-container-registry
[cicd-buzz] $ heroku container:login
[cicd-buzz] $ heroku container:push web
[cicd-buzz] $ heroku ps:scale web=1
```

After these commands you should be able to access the app at the url reported by the heroku create command.

Note that the heroku container:push web command, pushes the same container to the Heroku platform as you’ve pushed to the Docker Hub registry.

To automate the process of deploying each build of the master branch of our project, add the following to a file called ‘deploy_heroku.sh’ in the directory ‘.travis’:

```
#!/bin/sh
wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh
heroku plugins:install heroku-container-registry
docker login -e _ -u _ --password=$HEROKU_API_KEY registry.heroku.com
heroku container:push web --app $HEROKU_APP_NAME
```

Also, add the following line to your ‘.travis.yml’ file:
```
after_success:
  - sh .travis/deploy_dockerhub.sh
  - test "$TRAVIS_BRANCH" = "master" && sh .travis/deploy_heroku.sh
```

And finally add 2 more environment variables to your repo at Travis CI. You can find the Heroku API key under your Heroku ‘Account Settings’. The Heroku App name is the name reported by the heroku create command.

Commit and push these changes to GitHub. A new Docker image should now be pushed to both Docker Hub and Heroku after the build succeeds.

### Step 10: CI/CD FTW!

Now that we have a modern development pipeline up and running, the fun of shipping functionality in short iterations and small increments starts. Let’s say we want to make the landing a bit more attractive. A typical workflow to do that looks like:

1. Start by creating a new issue for the feature: https://github.com/robvanderleek/cicd-buzz/issues/1
2. Create a Git feature-branch for this ticket: https://github.com/robvanderleek/cicd-buzz/tree/issue-1
3. *Coding magic happens here*
4. Keep an eye on the feedback from Travis CI and Better Code Hub: https://github.com/robvanderleek/cicd-buzz/commits/issue-1
5. Check a running instance of your app by locally running the latest Docker image:docker run --rm -p5000:5000 -it robvanderleek/cicd-buzz:issue-1 You can also share this container with others.
6. If you’re satisfied with the new feature, open a Pull Request and your code is ready to be shipped by the CI/CD pipeline to production: https://github.com/robvanderleek/cicd-buzz/pull/2
