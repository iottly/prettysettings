It allows to save and retrieve configuraton settings from a json file.

- usage: 
  `from prettysettings import Settings`
  `settings = Settings(defaults = {'option1': 1, 'option2': 'myoption'}, filename='./settings.json')`
  `print(settings.option1)`

- if any of the settings is found in env variables than it is overridden

- override order is:
  - first read from defaults
  - then overrides from file
  - last overrides from env variables

- type:
  - defaults dict is used as reference for type parsing when loading from env variables

- keys:
  - defaults dict is used as reference to load keys from file and from env variables
  - a key found in the file (or env vars) which is not present in the defaults is discarded