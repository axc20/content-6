# Python

- [python online compiler](https://www.programiz.com/python-programming/online-compiler/)
- [pipenv](https://pypi.org/project/pipenv/)
- [pyenv](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md)

```shell
# Mac M1
arch
which brew
brew --version
python --version

brew install pyenv
echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
brew install openssl readline sqlite3 xz zlib
pyenv install 3.9.6
pyenv versions
pyenv global 3.9.6
which python
python --version
brew install pipenv

pip --version
pipenv --version
pipenv --help

# list all globally installed packages
pip list
# uninstall globally installed packages | SO 11248073
pip uninstall -y -r <(pip freeze)
```

## pipenv, Pipfile, Pipfile.lock

Pipenv handles the virtual environment. Pipenv uses pyenv to install your desired version in a virtualenv just for that project.

```shell
brew install pipenv pyenv
cd my_cool_project
pipenv --python 3.9.6
pipenv run python3 my_cool_project.py

# in project folder:
pipenv install
pipenv install <package>
pipenv install --dev <package>
pipenv uninstall <package>
pipenv shell # activate the virtual env
exit
```

[pipenv guide](https://realpython.com/pipenv-guide/)

```shell
# spawn a shell in a virtual env
pipenv shell

# install packages
pipenv install flask==0.12.1
pipenv install numpy
pipenv install pytest --dev

# lock env -> create/update Pipfile.lock
pipenv lock

# once you get code and Pipfile.lock in prod env
# tell pipenv to ignore Pipfile and use what is in Pipfile.lock
pipenv install --ignore-pipfile

# in dev env
pipenv install --dev

# show dependency graph (or reversed tree)
pipenv graph
pipenv graph --reverse
```

The `Pipfile` intends to replace `requirements.txt`. It should convey the top-level dependencies your package requires.

- `[dev-packages]`: for development-only packages
- `[packages]`: for minimally required packages
- `[requires]`: for other requirements, like a specific version of Python

`Pipfile.lock` enables deterministic builds by specifying exact requirements for reproducing an env. It contains exact versions for packages and hashes to support more secure verification.

```shell
# open a 3rd party package in your default editor
pipenv open flask

# run a command in virtual env without launching a shell
pipenv run <command>

# check for security vulnerabilities in your env
pipenv check

# uninstall a package
pipenv uninstall numpy

# wipe all installed packages from your virtual env
pipenv uninstall --all
pipenv uninstall --all-dev

# find out where your virtual env is
$ pipenv --venv

# find out where your project home is
$ pipenv --where
```

Pipenv supports automatic loading of environmental variables when a `.env` file exists in the top-level directory. When you `pipenv shell` to open the virtual env, it loads your env variables from the file.

```
# .env
SOME_ENV_CONFIG=some_value
```

## API Frameworks

- https://rapidapi.com/blog/best-python-api-frameworks/
- requests, faster_than_requests - client library
- flask, FastAPI, Eve - micro framework
- Django - full stack framework
