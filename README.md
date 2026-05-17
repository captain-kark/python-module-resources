# python-module-resources

Import non-python files in a project directory as python namedtuple objects.

If you've ever worked in node, you might be familiar with a language feature which allows you to pull in a json file as an imported module.

```node
import { dataStuff } from 'myProjectResources/jsonFiles'

dataStuff.contents === "A rich javascript object"
```

With `module_resources`, you can achieve something similar in python.

```py
from my_project_resources.json_files import data_stuff

data_stuff.contents == 'A python namedtuple instance'
```

## Getting Started

To use this in your own project, install with pip.

```
pip install module-resources
# supports yaml files too
pip install module-resources[yaml]
```

Create a module location in your project where your desired importable resource files will live. I will use [this project as an example](./module_resources/examples).

```
mkdir module_resources/examples/json/
touch module_resources/examples/json/__init__.py
```

Your module's [`__init__.py` file does the heavy lifting](./module_resources/examples/json/__init__.py).

```py
from module_resources import JsonModuleResource
__all__ = JsonModuleResource(__name__, __file__).intercept_imports()
```

From there, move the resources you'd like to import as python objects into that directory.

```
tree ./module_resources/examples/json/
module_resources/examples/json/
├── __init__.py
└── logging_config.json

0 directories, 2 files
```

You can now import some logger object's configuration json as a python namedtuple. It has a few interesting properties about it that will hightlight some caveats about this tool.

```
> python
Python 3.7.2 (default, Jan 14 2019, 16:30:40)
[Clang 10.0.0 (clang-1000.10.44.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
```

```py
>>> from module_resources.examples.json import logging_config
>>> logging_config.formatters.simple
json(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
```

You may refer to any valid namedtuple property by using dot-attribute notation.

```py
>>> logger.formatters.simple.format
'%(asctime)s - %(name)s - %(levelname)s - %(message)s'
```

Yaml configuration files work as well, [with the `module_resources.YamlModuleResource` class](./module_resources/examples/yaml/__init__.py).

```py
>>> from module_resources.examples.yaml import logging_config as yaml_logging_config
>>> yaml_logging_config.formatters
yaml(simple=yaml(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'))
```

To turn this object into a dictionary, pass it to `dict()`.

```py
>>> import logging.config
>>> logging.config.dictConfig(dict(yaml_logging_config))
>>> logging.getLogger('test').info('testing')
```

```
2019-07-23 11:43:57,296 - test - INFO - testing
```

## Caveats

There are some caveats to be aware of.

#### Valid vs. invalid namedtuple field names

```py
>>> from module_resources.examples.json import logging_config
>>> logging_config.loggers.__main__
```

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'ImportableFallbackDict' object has no attribute '__main__'
```

In namedtuple fields, `__main__` is not a legal name. It gets exposed as a dictionary type, `ImportableFallbackDict`.

```py
>>> logging_config.loggers['__main__'].level
'INFO'
```

It does store valid property names in that dictionary as namedtuples, however.

#### Exporting only a portion of a file

You may export from the top level namespace of a resource.

```py
>>> from module_resources.examples.json.logging_cofig import formatters
>>> formatters.simple.format
'INFO'
```

You can't export arbitrarily deep from a json or yaml file. Only one level deep is supported.

#### Type hints for module resources

Using this tool is going to make pylint unhappy.

```py
from module_resources.examples.json import logging_config

import logging.config
logging.config.dictConfig(dict(logger))
```

The above code is going to raise complaints from pylint.

```
pylint: Unable to import 'module_resources.example.json.logging_config' ("import-error") [E0401]
```

This tool leverages highly dynamic features of the python language to accomplish its work, and as such properties and types won't be available until runtime. You will need to include exceptions for these objects in your codebase if you want to maintain a high lint score while using this tool.

# Development

A Linux or Mac environment is assumed for development, running python version 3.

### Prerequisites

```
make preqres
```

This script will help ensure you have the necessary tools for developing against the project locally.

### Installing

```
make virtualenv
. .venv/bin/activate
```

Run the tests after installing the dependencies.

## Running the tests

```
make tests
```

### And coding style tests

```
make lint
make mypy
make bandit
```

## Deployment

Open a pull request against the master branch. Travis-CI will publish a preview version after all tests pass. You can install this preview version of your branch build from test.pypi.org.

```
pip install -i https://test.pypi.org/simple/ module-resources==0.0.${TRAVIS_BUILD_NUMBER}
```

Note that if you open a pull request from a fork, this step won't run.

To publish a new official version, tag a commit and push it up to the master branch.

```
git checkout master
git pull origin master
# examples of preparing a new tag for release
make tag-patch # also accepts: tag-minor, tag-major
git push origin --tags
```

Note that you must create and push tags from the master branch only. Tags found in pull requests won't do anything.

## Contributing

Small pull requests are fine to submit directly as pull requests. Larger changes should first be submitted as issues and mapped out before starting work on the proposal.

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) for details on the code of conduct, and the process for submitting pull requests.

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags).

## Authors

See the list of [contributors](./CONTRIBUTORS.txt) who have participated in this project.

This list can be updated with `make contributors`.

## License

This project [is licensed under the MIT License](./LICENSE.md).

## Acknowledgments

* @PurpleBooth for [the README.md](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2) and [the CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) template files.
