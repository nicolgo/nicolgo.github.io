---
title: "Managing Python Projects"
date: 2022-06-09T08:19:16Z
hidden: false
draft: false
tags: ["Python","software"]
keywords: ["Python","software"]
description: 
slug: ""
---

## Introduction
The article is the summary of this [course](https://www.linkedin.com/learning/managing-python-projects/managing-python-day-to-day?autoplay=true).

We need some structure to work together effectively. 
  
A good process can help us avoid mistakes. Here are some steps:
  - Automation: machines are less prone to make mistakes than humans.
  - Checklists: Checklists contains best practices and eliminated error that comes from forgetting things. which can refer "The Checklist Manifesto"
  - Knowledge transfer

We can pack and reuse this code in other projects by writing code in an established way.
## Directory Structure

### Overview

Here's a folder structure for a python project:
```
project/
|- src/
|  |- __init__.py
|  |- project.py
|- test/
|  |- test_project.py
|- LICENSE.txt
|- Makefile
|- README.md
|- requirements-dev.txt
|- requirements.txt
|- setup.py
```
The details can be seen [here](https://github.com/353solutions/nlpy)

### README.md
For a project, it is very common that write ReadMe in Markdown format to provide information about the project. It should include these:
- short description of your project
- how to use it with examples
- how to make contributions to it
### \_\_init__.py
`__init__.py` tells Python that a directory can be imported as a package. It should contain these pieces of information:
1. docstring that consists of an elevator page and some example usage
2. version number: 
    `major_number.minior_version.patch_level`
3. import all the functions and classes from other sub-models or without additional code
### Tests
Firstly, put tests in a separate directory to ensure they do not run in production. Tests are necessary, and it also serves as commendation. The tests directory mostly contains Python files, but it can also contain some auxiliary data.
### Makefile
People tend to forget things like test steps. The best cure for this is automation. Once you write all the steps to run the tests in a script, documented and ready to execute. Makefile is a good tool to support task automation.
### setup.py
`setup.py` is Python's way of defining a project. Firstly, we import `setup` function. Next, we read the description from the readme file and read requirements from `requirements.txt`. Finally, we call the setup function.
## Dependency Management
### Package managers
Managing packages is not an easy task. Package managers like Pip, Poetry, Conda, Pipenv, and others will install third-party packages and their dependencies. For Pip, it always installs the latest version of a package, but installing a package with a specific version will help to reproduce the same environment in the future. Writing this in `requirements.txt` is useful.
### virtualenvs
A python interpreter can work with only one version of a package at a time. Creating a project-specific virtual environment to isolate the python installations is a good practice.
### Production vs. Development
The requirements used for production and development may be different. For example, production does not need packages for tests like Pytest. The common practice is to have separate requirements files, one(`requirements.ext`) for production and another one(`dev-requirements.txt`) for development. Another issue is that depending on the operating system. We might need different packages. The common practice here is to specify only top-level dependencies and let people decide what you need.

We can use `Makefile` to install packages for different requirements files. Here is an example:
```
env:
	# Create venv directory if not exist
	test -d venv || virtualenv venv
	./venv/bin/python -m pip install -r requirements.txt

dev-env: env
	./venv/bin/python -m pip install -r requirements-dev.txt
```
The `venv` rule says to create a virtual environment and to install only the requirements needed from production. The `dev-venv` rule relies on the `venv`, meaning it's going to execute first the `venv` rule and then add the development requirements
## Testing
### How to test
Testing is important. It validates our code and guards us against breaking one part of the code when changing another. We have no time to implement all kinds of tests such as out-of-memory, crash, fuzz, regression, and much more. The common practice is to focus on the test that brings the most value. The following test types usually have the best deal:
1. Integration, check the connection between subsystems and connection with external systems.
2. Regression, check if running code returns a known solution with known data.
3. Fuzzing, generating random data, and throwing it at the code.
4. Linters, are the static checkers that find common issues without running the tests.
5. Unit, check the code in isolation for valid output.
Finally, we should focus our effort on the tests that bring the most value based on the test result.

We never have enough tests. Tests take time and effort to maintain. The major factor in how much test is the cost of error. The higher the cost, the more test we should write.
### Pytest
A detailed guide can be found on the homepage of `pytest`.
### Test items

## Development Process
There is no true process; suitable is the best. There are several common practices, such as source control, issue tracking, feature branches, code review, and retrospective.
### Source control
Git.
### Issue tracking
Every piece of work you do should be tracked. It can be a back fix, a feature to implement, or a maintenance task. The seven issue-tracking systems, such as Jira, Trello, Asana, and more. A good issue should have a good title and a good description. Using a template is a good choice.
### Feature branches
A branch is a part of the development path on the same code base. A feature branch is a part development path that is dedicated to a specific feature or an issue. Once you start work on an issue, create a feature branch for it and work on this branch. A nice convention is to call the branch with the name of the feature.
### Code Review
Code reviews are effective in decreasing bugs. It is also a great way to share knowledge.
### Retrospective
A retrospective, sometimes called postmortem, is a meeting you do at the end of a period or a sprint to discuss how you can improve. Retrospectives are very effective. Sadly, people tend to skip them, and they have a low priority in many teams. You should always allocate time for retrospectives. You can do retrospectives in a physical meeting or in an asynchronous way in a chat room or a shared document. I highly recommend documenting these retrospectives since they are common knowledge and represent organizational memory. Retrospectives can get long and tedious. Here's a template for making them short and effective. Everyone writes or talks about three things that went well, and we should keep three things that didn't go well and how to improve them. During the meeting, if there are things to do, known as action items, make sure to add them to your issue tracking and allocate time for them.
## Conclusion
Structure our code better, write down your dependencies, and write better tests. In time, we will find our code is of better quality.
