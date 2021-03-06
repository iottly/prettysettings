# License

Copyright 2015 Stefano Terna

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

# prettysettings
prettysettings is a minimal library which allows to save and retrieve configuraton settings from a json file and from env variables.

## Override:
Override order is:
- first read from defaults
- then override from file
- last override from env variables

## Keys:
- defaults dict is used as a reference to load keys from file and from env variables
- if a key is found in the file (or env vars) which is not present in the defaults it is discarded

## Types:
- defaults dict is used to enforce type when loading from file or env variables
- this ensures that env variables, which must be strings, are parsed to the type used in the default dict
- parsed types are: `bool`, `int`, `float`, `str`
- any type `T` for which a string representation exists, so that `T(string)` is the expected object is supported
- hence if `defaults = {'a': 1}` while the file contains `{'a': '2'}`, the result will be `settings.a == 2` (`int`)
- whereas if `defaults = {'a': 1}` while the file contains `{'a': 'string'}`, the Settings' constructor will raise an exception

## Computed settings Hooks
After reading items from defaults, file, env, the consrtuctor can call hoooks to add custom computed items to the settings object.
- This can be handy to produce new items which are mere functions of the stored ones.
- Or it can be useful to add to the settings object complex configuration items (i.e. not serializable to strings).
- Computed settings are not persisted when updating the settings object.

# Usage
## basic usage, just defaults: 
```python
from prettysettings import Settings
settings = Settings(defaults = {'option1': 1, 'option2': 'myoption'})
print(settings.option1) # output 1
print(settings.option2) # output 'myoption'
```
## load and update from and to file

Say you have `settings.json` containing:
```json
{
    "option1": 2,
    "option2": "my_new_option"
}
```
then:
```python
from prettysettings import Settings
settings = Settings(defaults = {'option1': 1, 'option2': 'myoption'}, filename='./settings.json')
print(settings.option1) # output 2
print(settings.option2) # output 'my_new_option'

# whereas
settings = Settings(defaults = {'option1': 1, 'option2': 3}, filename='./settings.json')
# raises an exception for type incompatibility among option2 in defaults and in file.

```

About filename:
- if filename does not exists it is skipped and Settings object is created with defaults
- this allows to create settings from defaults > updating them > persisting them to file even if it didn't exist at creation time

Say `settings.json` does not exists:
```python
from prettysettings import Settings
settings = Settings(defaults = {'option1': 1, 'option2': 'myoption'}, filename='./settings.json')

settings.update( {'option2': 'my_new_option'}, persist=True)

```
will produce `settings.json` containing:
```json
{
    "option1": 1,
    "option2": "my_new_option"
}
```

## Computed settings Hooks

```python
from prettysettings import Settings

cshooks = {
    'computedoption': lambda settings: 'hey this is computed with {}!!'.format(settings.option2)
}

settings = Settings(defaults= {'option1': 1, 'option2': 'myoption'}, computed_settings_hooks=cshooks)
print(settings.option1)          # output 1
print(settings.option2)          # output 'myoption'
print(settings.computedoption)   # output 'hey this is computed with myoption!!'
```

**Hint**: if you have more hooks with dependencies with one another, just wrap whem within an OrderedDict to be sure they will be executed in the correct order:

```python
from collections import OrderedDict
from prettysettings import Settings

cshooks = OrderedDict([
    ('computedoption', lambda settings: 'hey this is computed with {}!! '.format(settings.option2)),
    ('computedoption2', lambda settings: settings.option1 * settings.computedoption)
])

settings = Settings(defaults= {'option1': 2, 'option2': 'myoption'}, computed_settings_hooks=cshooks)
print(settings.option1)          # output 2
print(settings.option2)          # output 'myoption'
print(settings.computedoption)   # output 'hey this is computed with myoption!!'
print(settings.computedoption2)   # output 'hey this is computed with myoption!! hey this is computed with myoption!!'
```

