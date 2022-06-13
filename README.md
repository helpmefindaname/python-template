# python-template

Python template with opinionated setup of my personal preferences.

## adoption

* [ ] Create a git repository based on this template
* [ ] In `setup.py` adjust the `author` `author_email` `name` and `url`
* [ ] Rename the folder `python_template` to the project name
* [ ] depending on the publishing set `is_package` and `publish_docker` accordingly.
* [ ] if you use `publish_docker` set secrets for `DOCKER_REGISTRY` `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`. You may want to also adjust the name of the tag that is used for publishing.
* [ ] if you use `is_package` set secrets for `PYPI_API_TOKEN`. You might also want to specify a repository if you don't want to push to pypi. 
* [ ] Create a proper description of the project in the `README.md` file. Including changing the title.
* [ ] Review the other sections of the readme and adjust those to the actual settings.
* [ ] Delete this section of the readme.

**Note:** You can use this list as a checklist by replacing each `[ ]` with `[x]` after finishing it.

## Installation

To install this project, first clone it from git.
Then install it via pip: 
```
pip install .
```
If you want to work on this project, create a conda environment or a virtual environment and install it in the editable mode and add the `dev` extra:
```
pip install -e .[dev]
```

## Features

### CI-CD
CI-CD for automated testing and deployment.
Tests are tested on docker and on python 3.10. If the project should be published as package, it will be also tested on python 3.7, python 3.8 and python 3.9.
Additionally, a docker image can be pushed to a docker registry.

### Snapshot testing
There is an [https://martinahindura.medium.com/snapshot-testing-in-data-science-f2a9bac5b48a](Article) about snapshot testing in DS/AI projects, describing it's origin from the frontend testing.
I find it useful to test machine learning models and ensure that some code changes won't change the system (or if so display them).
In general:
You can write tests as in the following:
```python
import json

def test_ml_model(testdata, test_snapshots):
    input_path = testdata("model_in.json")
    output_path = testdata("model_out.json")
    with input_path.open("r", encoding="utf-8") as f:
        input_data = json.load(f)
    
        model = MyModel.load("model")

    test_snapshots(model.predict(input_data), output_path)
```
It will require you, to create a file `model_in.json` in the `tests/data/` folder.
After that, you can test it by running:
```
pytest
```
At first it will fail, because it hasn't created any output.
You can simply create them by running the tests again with the `--gen-snapshots` parameter.
```
pytest --gen-snapshots
```
This will create the `model_out.json` file in the `tests/data/` folder.
After that, running pytest should succeed:
```
pytest
```
I adopt the following practice:
If you changed something, you can run the tests to see if it also changes the output. Depending on your changes, this might be expected.
For example updating the production model, should always yield different predictions.
I simply **recreate** the snapshots and use **git diff** to review the changes. That way, updating the model is not only reviewed by looking at an accuracy score,
but also review some examples qualitatively.
As the changes are also part of the pull request, a reviewer can also look at them.

This ensures that some standard mistakes will be catched early on, e.g.:
* Quantizing a model without testing the quantized version. (Quantization could heavily reduce accuracy if the wrong layer is quantized)
* Not using `model.eval()` when running predictions, so dropout is activated and reduces prediction quality.

### Collection of diverse code quality packages

usage of diverse plugins for pytest, such that mypy, flake8, black checks, isort checks, etc. are run when running pytest.

### deployment via tags.

To deploy a branch (depending on you CI-CD settings, this creates a docker image or publishes a package),
it is enough to create a tag within the `v#.#.#` schema and use semantic versioning.
There is no need to edit the `setup.py` or do any adjustment to the code.


## features that I refrained from adding.

There is a list of possible additions, that I do not like and therefore didn't add.


### poetry

While [poetry](https://python-poetry.org/) is a easy way of managing dependencies, I refrain from it, as I made bad experiences with venv, which is used implicitely.
When I started with Python and DL, I had to use anaconda to install pytorch on windows. In general there are often libraries that are easier to install via anaconda than with pip.
So I prefer a setup where you can freely create your environment on your own.
For this having requirements.txt is crucial, as it allows me to install some dependencies with a different usage.

### pre-commit hooks for formatting

I refrain from using pre-commit hooks for code linting & formatting.
Most pre-commit hooks are set up, to use their own configuration / version. This can lead to unwanted behaivours. For example having locally formated with by running `black .`
can then be reverted by the pre-commit hook, if the version of black is different or any configuration (e.g. the max line length) diverges.
Ideally you can commit your work at any time (in case of fire, at the end of the work day, etc.) without anything trying to keep you away from it. Read about [commit early, commit often](https://marijn.huizendveld.com/blog/commit-early-commit-often-even-when-the-tests-are-failing). It is more important that you commit, than that it is working.
Ideally you create the habit of testing your code before you commit and therefore also get reminded of any code that is not formatted perfectly.
With that habit, you don't need pre-commit hooks and if you don't have that habit, I doubt that pre-commit hooks will help you that much, as tests might fail anyways.
I have seen quite some anti-patterns with pre-commit hooks such as
* running them as part of CI-CD (Imagine setting up DVC and then have the pipelines start downloading your 2GB model that you don't need for tests, due to this setup.)
* trying to run them via pytest.
* using them in combination with a doc-test and instead of skipping pre-commit hooks adding empty worthless docs which weren't fixed later.


## features on my todo list

### multi stage docker build
nothing to add here

### requirements-freeze
I used to use a setup that involves a second docker file together with a docker-compose.yaml that runs containers that simply
* installs `requirements.txt`
* freezes it out to a `requirements-frozen.txt`
the `requirements-frozen.txt` will then be used within the original docker file.

The idea of it is simple:
If you run a deployment via docker images, you want to have the specific dependencies that were used for testing them.
You don't want to have any other dependencies that might come due to using a different OS or having something different in your setup.
Hence the only way to ensure the exactly same setup, is by using a dockerfile with the exact setup.
The CI-CD system will test using the frozen dependencies and therefore ensures, that the developers won't just get it work on their machine.
However they are still able to install whatever they want to make it work on their machine too.
