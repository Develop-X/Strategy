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

Full disclosure: I’m one of the developers on the Better Code Hub team.

Update August 2017: Eran Barlev from codefresh.io published a short video on LinkedIn demonstrating how Codefresh can replace some of the building blocks described in this tutorial. Since no two software products are the same, development pipelines can vary too. Be sure to check out Eran’s video to see an alternative implementation of a modern CI/CD pipeline.

Step 1: Write a little buzz generator
We need a small piece of software that will travel through all phases of the pipeline, from your laptop to the cloud. In our case this piece of software is a little CI/CD buzz generator program written in Python.

Create a new directory (let’s say we call it ‘cicd-buzz’). Inside this directory create another directory called ‘buzz’ and in this directory save the snippet below to a file called ‘generator.py’.


generator.py
Also create a new empty file called ‘__init__.py’ in the same directory. Your project structure should look like this now:

cicd-buzz/
  buzz/
    __init__.py
    generator.py
You should be able to run this Python script from the command line inside the ‘buzz’ directory:

[cicd-buzz/buzz] $ python generator.py
End-To-End Devops Enormously Boosts Continuous Testing
Try it a couple of times, it’s fun:

[cicd-buzz/buzz] $ python generator.py
Complete Continuous Improvement Enormously Improves Devops
[cicd-buzz/buzz] $ python generator.py
Modern Devops Remarkably Improves Continuous Testing
Step 2: Add automated tests
Continuous delivery pipelines only makes sense when you have a significant amount of automated tests that prevents you from continuously shipping broken software. To get a proper set of unit-tests for our buzz generator create a new directory called ‘tests’ in the root of your project directory and save the the snippet below to a new file in the ‘tests’ directory. Call this file ‘test_generator.py’:


test_generator.py
To run the tests we’ll be using the ‘pytest’ framework. To install pytest we’ll be using a Python Virtual Environment (‘virtualenv’). Don’t worry, that’s easier done than said. First, make sure you have virtualenv installed so you can execute the following command inside your project directory:

[cicd-buzz] $ virtualenv venv
This should create a new directory venv. To start using this environment type:

[cicd-buzz] $ source venv/bin/activate
(venv) [cicd-buzz] $
Next, create a new file called ‘requirements.txt’ that lists the pytest dependency:

pytest==3.0.6
To download dependencies listed in the requirements file you’ll need to execute the ‘pip’ command:

(venv) [cicd-buzz] $ pip install -r requirements.txt
After you’ve completed all steps above, the root of your project directory should look like this:

cicd-buzz/
  buzz/
  requirements.txt
  tests/
  venv/
Inside the virtual environment you can now run the unit-tests in the ‘test_generator.py’ file: