## Project structure

Here is a basic structure generated by the
[AnyBlok/cookiecutter-anyblok-project][cookiecutter] as you
learned in the how to [Set up your own project](./02_cookiecutter.md).


```bash
├── app.cfg                      # Configuration file used in production environment
├── app.dev.cfg                  # Configuration file used for development
├── app.test.cfg                 # Configuration file used to run tests
├── CHANGELOG.rst                # Change log of the project
├── doc                          # Documentation root directory
│   ├── build                    # Directory where the output documentation(s) is built
│   ├── Makefile                 # Used to build the documentation
│   └── source                   # Source "code" documentation directory
│       ├── apidoc.rst           # Predefined apidoc chapter loading api blok documentation
│       ├── conf.py              # Sphinx configuration documentation
│       ├── index.rst            # Main documentation page
│       ├── _static              # Sphinx documentation assets
│       └── _templates           # Sphinx templates
├── LICENSE                      # Project license
├── Makefile                     # A Makefile: see make help to list available commands
├── README.rst                   # Project's README with basics project information
├── requirements.dev.txt         # Python package dependencies required for development
├── requirements.test.txt        # Python package dependencies required for running unittest
├── rooms_booking                # The python package directory where you develop your bloks 
│   ├── room                     # A blok directory
│   │   ├── __init__.py          # Blok definition
│   │   ├── model.py             # Python module with model that define an example class
│   │   ├── tests                # Test directory
│   │   │   ├── __init__.py      # Likes standard python, the __init__.py file!
│   │   │   ├── conftest.py      # py.test configuration file. (You will likely import ``anyblok.conftest``
│   │   │   ├── test_model.py    # File that test model.py methods
│   │   │   └── test_pyramid.py  # File that test view.py methods
│   │   └── views.py             # Python module to declare pyramid route components 
│   └── __init__.py              # Likes standard python, the __init__.py file!
├── setup.cfg                    # The python package setup.cfg file
├── setup.py                     # The python package setup.py file
├── tox.ini                      # A default tox config file
└── VERSION                      # Python package version file
```

An AnyBlok project follows the rules of [python package projects]
[doc_python_package], there are few additional requirements:

* You must add ``room`` blok in the ``bloks`` list 
``entry_points`` in your package manifest: the ``setup.py``
file:

```python
entry_points={
    'bloks': [
        'room=rooms_booking.room:Room'
        ]
},
```

* Then you must declare your blok in ``rooms_booking/room/__init__.py`` by
  creating a python class that inherits from ``anyblok.blok.Blok``:

```python
"""Blok declaration example
"""
from anyblok.blok import Blok


class Room(Blok):
    """Room's Blok class definition
    """
    version = "0.1.0"
    author = "Pierre Verkest"
    required = ['anyblok-core']

    @classmethod
    def import_declaration_module(cls):
        """Python module to import in the given order at start-up
        """
        from . import model  # noqa

    @classmethod
    def reload_declaration_module(cls, reload):
        """Python module to import while reloading server (ie when
        adding Blok at runtime
        """
        from . import model  # noqa
        reload(model)

    @classmethod
    def pyramid_load_config(cls, config):
        """configure pyramid server route"""
        config.add_route("root", "/")
        config.add_route("example_list", "/example")
        config.add_route("example", "/example/{id}")
        config.scan(cls.__module__ + '.views')

    def update(self, latest):
        """Update blok"""
        # if we install this blok in the database we add a new record
        if not latest:
            self.registry.Example.insert(name="An example")
```

As you can see a python package can contain one or more bloks.

![bloks](../../static/package_anyblok_wms_base.png)

An Anyblok application project is a blok by itself that will have requirements
to some other bloks published in other python packages or within the same
package.


[cookiecutter]: https://github.com/AnyBlok/cookiecutter-anyblok-project
[doc_python_package]: https://packaging.python.org/tutorials/packaging-projects/