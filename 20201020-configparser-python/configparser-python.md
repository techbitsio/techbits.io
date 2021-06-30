<!--- META
title=Python: Create config files with configparser
publish_date=20201106
description=How to create and use config files using configparser for Python 3
header_image=python-code.jpg
tags=python
comments=2
-->

Configparser is a simple, built-in way to start using config files in your Python scripts and programs.

Imagine our config.ini file looks like this:

```
[DEFAULT]
site_ranking = 5
site_speed = 5

[example_section]
url = yourawesome.site
port = 443
site_ranking = 10
```

Getting started is a simple as:

```python
import configparser

config = configparser.ConfigParser()
config.read('config.ini')
```

Now we can access items much like we're using a dictionary, e.g. `config['example_section']['url']` would return `yourawesome.site`. You can address one section directly (useful if you only have one section in the file):

```python
>>> es = config['example_section']
>>> es['site_ranking']
10
```

The `get` method can also be used, which allows for fallback values: `config.get('example_section', ‘not_port', fallback='9000')` would return `9000`.

The `DEFAULT` section provides values when keys do not exist in our `example_section` — even with a fallback value, the default value takes precedence:

```python
>>> config.get('example_section', 'site_speed', fallback='10')
5
```

The return values will always be strings, unless methods like `getint()`, `getfloat()`, `getboolean()` are used. For booleans, values in the config file can be: 0, 1, no, yes, false, true.

Things to note:

- Keys are not case sensitive, but values are
- Values are assigned to keys with `:` or `=`
- DEFAULT values have precedence over fallback values
- Strings are returned by default for all values

Please let me know if you have some useful 'getting started' tips for Configparser!

*Header image by [Johnson Martin](https://pixabay.com/photos/code-programming-python-1084923/)*
