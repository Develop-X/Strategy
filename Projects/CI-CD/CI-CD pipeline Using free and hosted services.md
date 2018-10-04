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
