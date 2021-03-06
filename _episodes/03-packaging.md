---
title: "Packaging Code for Release"
teaching: 0
exercises: 90
questions:
- "How do we prepare our code for sharing as a Python package?"
- "How do we release our project for other people to install?"
objectives:
- "Describe the steps necessary for sharing Python code as installable packages."
- "Use Poetry to prepare an installable package."
- "Explain the differences between runtime and development dependencies."
- "Upload an installable Python package to a package index."
keypoints:
- "Poetry allows us to produce an installable package and upload it to a package index."
- "Making our software installable with pip makes it easier for others to start using it."
- "For complete control over building a package, we can use a setup.py file."
---

## Where do Packages Come From?

All the dependencies we've installed previously with `pip` were written by someone else and uploaded to a **Package Index / Repository**.

In this session we'll be looking at two different methods for building an installable package from our code.
The first, using a tool called **Poetry**, is the simpler of the two methods, so we'll walk through the complete packaging process and end up with a package uploaded to the PyPI test site.
The second method, using **setup.py**, is the more traditional method and gives us full control, but is more complicated.

## Packaging our Code with Poetry

### Installing Poetry

Before we start this section, let's make sure again that we don't have any virtual environments currently activated.
If we don't currently have a virtual environment activated this will give us an error message or tell us to use `source deactivate` - that's fine.

~~~
deactivate
~~~
{: .language-bash}

The recommended install method for Poetry is similar to the method we used for pyenv.
This time it's a Python script we need to download and run:

~~~
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python3
~~~
{: .language-bash}

While Poetry is installing, it lets us know that we might have to run an extra command before we can use it.
Once it's finished installing, the simplest thing to do here is to just close our terminal and open another one.
Then the changes that Poetry makes should have been applied automatically for us.

To test, we can ask where Poetry is installed:

~~~
which poetry
~~~
{: .language-bash}

~~~
/home/sabsr3/.poetry/bin/poetry
~~~
{: .output}

If you don't get this output, then we should try activating it manually and checking again:

~~~
source $HOME/.poetry/env
which poetry
~~~
{: .language-bash}

~~~
/home/sabsr3/.poetry/bin/poetry
~~~
{: .output}

### Setting up our Poetry Config

Poetry uses a **pyproject.toml** file to describe the build system and requirements of the package.
This file format was described in [PEP 518](https://www.python.org/dev/peps/pep-0518/) to solve problems with bootstrapping (the processing we do to prepare to process something) packages using the older convention **setup.py** files and to support a wider range of build tools.

First let's make sure we're in the right place:

~~~
cd
cd 2020-se-day4/code
~~~
{: .language-bash}

Because we're going to use Poetry to manage our dependencies for us, we should deactivate, remove and remake our virtual environment to make sure it's clean and also make sure we're not using the risky Python 3.10 development version from last session.
Remember that when we do `deactivate` we might get an error or warning if we weren't already in a virtual environment - this is fine.

~~~
deactivate
rm -rf venv .python-version
python3 -m venv venv
source venv/bin/activate
~~~
{: .language-bash}

Now we're ready to begin.

To create a `pyproject.toml` file for our code, we can use `poetry init`.
This will guide us through the most important settings - for each prompt, we either enter our data or accept the default.
For the package name, make sure this has some unique identifier in it so it doesn't match the package name of anyone else - I've used my name here, you could use your name, or some random text.
This is because, if we want to upload it to the test version of PyPI at the end, it will need to be globally unique.
Note that, usually when we're naming a Python installable package, we use hyphens to separate words.

When we get to the questions about defining our dependencies, we'll answer no, so we can do this separately later.

~~~
poetry init
~~~
{: .language-bash}

~~~

This command will guide you through creating your pyproject.toml config.

Package name [example]:  poetry-project-jgraham
Version [0.1.0]: 0.1.0
Description []:  Example project for using Poetry to build packages
Author [None, n to skip]: James Graham <J.Graham@software.ac.uk>
License []:  MIT
Compatible Python versions [^3.8]: ^3.8

Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your development dependencies interactively? (yes/no) [yes] no
Generated file

[tool.poetry]
name = "poetry-project-jgraham"
version = "0.1.0"
description = "Example project for using Poetry to build packages"
authors = ["James Graham <J.Graham@software.ac.uk>"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry>=1.0.0"]
build-backend = "poetry.core.masonry.api"


Do you confirm generation? (yes/no) [yes] yes
~~~
{: .output}


### Project Dependencies

Previously, we looked at using a `requirements.txt` file to define the dependencies of our software.
Here, Poetry takes inspiration from package managers in other languages, particularly NPM (Node Package Manager), often used in JavaScript.

Tools like Poetry and NPM understand that there are two different types of dependency: runtime dependencies and development dependencies.
Runtime dependencies are those dependencies that need to be installed for our code to run, like NumPy.
Development dependencies are dependencies which are an essential part of your development process for a project, but are not required to run it.
Common examples of developments dependencies are linters and test frameworks, like Pylint or Pytest.

When we add a dependency using Poetry, Poetry will add it to the list of dependencies in the `pyproject.toml` file, add a reference to it in a new `poetry.lock` file, and automatically install the package into our virtual environment.
The `pyproject.toml` file has two separate lists, allowing us to distinguish between runtime and development dependencies.

~~~
poetry add numpy
poetry add --dev pylint
~~~
{: .language-bash}

These two sets of dependencies will be used in different circumstances.
When we build our installable package and upload it to a package index, Poetry will only include references to our runtime dependencies.
This is because someone installing our software through a tool like `pip` is using it, but probably not developing it.

In contrast, if someone downloads our code from GitHub, with our `pyproject.toml` and installs our project using that, they will get both our runtime and our development dependencies.
If someone is downloading our source code, that suggests that the intend to contribute to the development of it, so they'll need all of our development tools.

### Packaging Our Code

Next, we need to make sure that our code is organised in the recommended Python code package structure.
This is the package (yes, we use the same word to mean two different things...) structure that we encountered in the refactoring section - a directory containing an `__init__.py` and our Python source code files.

We've provided an example using the academic model in the code directory for today, but you'll have to rename this package so that it matches the name you told Poetry about, but with underscores instead of hyphens.
By convention installable package (the type we install with `pip`) names use hyphens, whereas code package (a directory of Python files) names use underscores.
While we could choose to use underscores in an installable package name, we cannot use hyphens in a code package name, as Python will interpret them as a minus sign.

~~~
mv poetry_project poetry_project_jgraham
~~~
{: .language-bash}

Once we've got out `pyproject.toml` configuration done and our code in the right structure, we can go ahead and build a distributable version of our software:

~~~
poetry build
~~~
{: .language-bash}

This should produce two files for us in the `dist` directory.
The one we care most about is the `.whl` or **wheel** file.
This is the file that `pip` uses to distribute and install Python packages, so this is the file we'd need to share with other people who want to install our software.

To install this wheel file, we can give it to `pip`:

~~~
pip3 install dist/poetry_project*.whl
~~~
{: .language-bash}

The star in the line above is a **wildcard**, that means Bash should use any filenames that match that pattern, with any number of characters in place for the star.
We could also rely on Bash's autocomplete functionality and type `dist/poetry_project`, then hit the <kbd>Tab</kbd> key.

### Sharing Our Package With The World

The final step in distributing our code is to upload it to a package index.
To help us test our packages, PyPI provides a test index at [https://test.pypi.org/](https://test.pypi.org/) that we can use.
Packages uploaded to the test index aren't accessible to a normal `pip install`, so no one's going to accidentally install our software yet.

Firstly, we need to create an account at [https://test.pypi.org/](https://test.pypi.org/).
Click the register link to the top right, and fill in your account details.
Once you've created your account, you may receive an email asking you to verify the account creation.

Now, we need to tell Poetry about our account on the test PyPI server.
When we enter the second of the following commands, Poetry will also ask us to enter our password:

~~~
poetry config repositories.testpypi https://test.pypi.org/legacy/
poetry config http-basic.testpypi <your username>
~~~
{: .language-bash}

> ## Security Alert!
>
> Since we're publishing our code for other people to use, we need to take care to do this securely.
> If someone gets access to our PyPI account, they could potentially upload a malicious version of one of our packages.
> This has happened with several JavaScript packages in the past.
>
> To improve our security, we should use an **API key / token** rather than a username and password.
>
> For more information, see the [PyPI FAQs](https://test.pypi.org/help/#apitoken).
{: .callout}

Finally, we're ready to go.
To publish the our software that we've been working so hard on, there's just one more command:

~~~
poetry publish -r testpypi
~~~
{: .language-bash}

If we now go to [https://test.pypi.org](https://test.pypi.org/) and search for our package name, we should find our newly published software.
We can even install this package ourselves using `pip`:

~~~
pip3 install -i https://test.pypi.org/simple/ <your package name>
~~~
{: .language-bash}

Note that there is a space between the URL and your package name in the line above.

After we've been working on our code for a while and want to publish an update, we just need to update the version number in the `pyproject.toml` file (using SemVer perhaps), then use Poetry to build and publish the new version.
If we don't increment the version number, people might end up using this version, even though they thought they were using the previous one.
Any re-publishing of the package, no matter how small the changes, needs to come with a new version number.

~~~
poetry build
poetry publish -r testpypi
~~~
{: .language-bash}

In addition to the commands we've already seen, Poetry contains a few more that can be useful for our development process.
For the full list see the Poetry [CLI documentation](https://python-poetry.org/docs/cli/).

If you've made it this far, congratulations, you've successfully published and installed a Python package!
Though it's becoming increasingly common for academic software to be shared under an open source license, not many people go this extra step and make their code installable so easily.
Remember that the easier we make it for people to use our code and get involved with the project, the easier it is for people to reproduce and build upon our work.
Funders are also starting to put greater emphasis on sharing the outputs of our projects as this increases the impact of our work and the impact of their money.

> ## Adding Some Detail
>
> Using the [Poetry documentation](https://python-poetry.org/docs/), investigate how we might go about adding more detail to our page on the package index website.
> We want people to be able to find our package and to be able to tell if it's going to be useful to them.
> What extra information can we add to the `pyproject.toml` file to help with this?
{: .challenge}

## What If We Need More Control?

Sometimes we need more control over the process of building our installable package than Poetry allows.
In these cases, we have to use the method that existed before Poetry - a `setup.py` file.
Because this is a Python file, we can use the full power of Python to describe how to setup our project.

One of the common cases where this is particularly useful is if our project has components in different languages.
For example, to speed up some of the core parts we might write some of our functions in C, then call these from our Python code.
Using a `setup.py` gives us the flexibility to handle building these components in different ways and bring them together at the end.

In the template repository for the mini-projects, we have an example of a basic, general purpose `setup.py` file.
You can find this file [https://github.com/SABS-R3/2020-software-engineering-projects-pk/blob/master/setup.py](https://github.com/SABS-R3/2020-software-engineering-projects-pk/blob/master/setup.py).

> ## Our Own Setup.py
>
> Compare this example `setup.py` file to the Poetry `pyproject.toml` file we created previously.
> At the bottom, in the arguments to the `setup` function, we have many of the same pieces of metadata.
>
> The [Python Packaging User Guide](https://packaging.python.org/) provides documentation on the details of packaging a project using `setup.py`.
> In this guide, they use Twine to upload the package to PyPI, instead of Poetry as we did previously.
>
> Using the example `setup.py` file and this documentation, can you produce a `setup.py` file for our Poetry project?
>
> Which configuration style do you prefer for projects like this one?
{: .challenge}

{% include links.md %}
