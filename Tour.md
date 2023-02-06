# A quick tour

This repo demonstrates how to build a CLI tool called `demo` using software dev best practices and useful tools.

## Getting started

Open a WSL prompt and note your python version for later `python3 --version`

Now clone the repository inside WSL into a work folder. I typically start dev code work in /projects:

```
cd /projects
git clone git@github.com:alastairtree/python-cli-devenv-and-ci-sample.git
cd python-cli-devenv-and-ci-sample
code .
```

This has opend the folder, Opened VS code. Now follow the prompt to reopen in dev container. This will take a few minutes the first time as it builds a VM with all your tools and dependencies inside it using docker,

Now run the app:

Open a terminal (CTRL + ').
Note how automatically you are in a python venv!

```
demo --help
```

We did not install anything, no setup for python, we did not think about versions - it just works.

Also note the python version is probably different from earlier `python3 --version` - this is because you are now inside a docker container

# Commands

Lets look at commands using Typer. Try a command: `demo hello`

Note how it gave a pretty error message "missing argument NAME".

Fix wih `demo hello Bob`. Typer makes it easy to make nice CLI tools and has lots of docs.

# Debugging

VS Code debugging is setup. Put a breakpoint in hello() in [main.py](src/main.py) and F5 (Run > Debugging). Note how you can inspect the name. Args have been passed based on config in [launch.json](.vscode/launch.json) so everything works out-the-box.

# Using Typer Arguments and Options

Now for arguments. Compare `demo goodbye Alice` and `demo goodbye --formal Alice`

Now look at the code in [main.py](src/main.py). See how simple this is? no argument parsing code. This is because we are using Typer - an OSS library on PyPy that was installed automatically using poetry (like pip!).

You can also use Typer Options con control command line args:

```
@app.command()
def goodbye(name: str, formal: bool = False):

# can also be written like this to provide defaults, help and and alternative forms

def goodbye(
    name: str,
    formal: bool = typer.Option(
        False,
        "--formal",
        "-f",
        case_sensitive=False,
        help="How polite would you like the greeting?"
    )
):
```

## Poetry - dependencies and builds

Our demo app is built on top of open source packages. List them using `poetry show` and see details in [pyproject.toml](pyproject.toml). New packages can easily be installed e.g. `poerty add numpy` - see your pyproject and lock files have been updated. Everyone else also gets the same packages because the manifest is checked in and they are distributed with the package when you share the tarball.

Look at the /dist folder after you build:

```
poetry build
```

See the tarball. Anyone can install that with pip (or better with pipx). See the [install.sh](install.sh) script for more info so sharing your app is possible.


## Automated testing

Testing is easy using pytest which is built-in and available on the terminal. Again pre-installed.

run 'pytest --verbose'

Look at [test_main.py](tests/test_main.py). The test tooling in VS code is also enabled by default, and supports debugging.

We also get code coverage information - run the the build script [build.sh](build.sh) and then open the results in a web browser

from WSL (not vscode terminal!), open the html file in windows:
```
sensible-browser htmlcov/index.html
```

## Using different versions of python

At the start you noted the version of python on your machine. We probably want to develop with a newer version - so we develop inside a docker container. We also want to be backwards conpatible with other versions so we can share with others on different versions.

The dockerfile installs pyenv and several python version for you automatically. You can add more easily by changing the docker file and rebuilding.

List the versions installed inside the docker container:

```
pyenv versions
```

You can change to a different version of python easily to check it works for a different version, and avoids messing up the python on your (wsl) host.

```bash
# you are probably already in 3.11
python3 --version

# if already in a env, get out
deactivate

# configure the local folder and poetry to use 3.10, restore dependencies and run the app
pyenv local 3.10
poetry env use python3.10
poetry shell
poetry install

# now in 3.10
python3 --version

# app still works!
demo hello world

# now restore the local folder and poetry to use 3.11
deactivate
pyenv local 3.11
poetry env use python3.11
poetry shell
poetry install

```

Also check out the build script: it builds multiple versions automatically:

```
./build-all.sh
# look in /dist folder - one folder per version automatically
```

## Continuous integration

Every commit is built and tested by GitHub Actions automatically against every version of python. See https://github.com/alastairtree/python-cli-devenv-and-ci-sample/actions.

You can see each version is handled automatically. Based on config in [ci.yml](.github/workflows/ci.yml)

Coverage reports and tarballs are attached to every build. Pull requests get tested automatically.

## Linting and formatting

Run lint and format tools on your code from the terminal as they are pre-installed. They run on save in vscode and on cli. You can disdable the autosave feature by removing editor.* in settings.json so you can run the tools on the cli as well:

```bash
# delete a colon in main.py and then run flake for nice errors
flake8 src

# delete whitespace in main.py and run
black src

# also keep multiple imports tidy
isort src
```
## Wrap up

Hopefully this showed you how to use vscode, docker, poetry, typer, pytest and github to build userfirndly and easy to maintain CLI apps.;
