schema sugar
==============

# Why?
Combine all data in one json-schema:
Doc, api, commandline are generated automatically : )
Validation will be done automatically.

# How it works?

It reads schema from json-schema then generate api view(flask) and commands.
 The doc is also generated by template.

# Feature
+ Almost everything defined by json-schema, can be dumped easily
+ RestAPI and cli generated or registered automatically by schema-definition
+ Default behavior for a resource defined by default, inherit and implement it
+ Call remote server in on package with http-request cli(to be implemented)
+ Validation and output filter, write your logic, no worry about validation
+ API Document can be generated automatically for each resource(to improve) 
+ Generate code for other language by template(not implement)

# Try it
Run:
```bash
git clone git@github.com:winkidney/schema-sugar.git

cd flask_cmd
sudo python setup.py develop

cd examples
sudo python setup.py develop
```

## WebAPI
```bash
# run web_server
rest-server
```
Then open your browser with 
```
# resource root
http://locahost:7001/api/disks/1

# resource doc
http://locahost:7001/api/disks/1/meta

# site map
http://localhost:7001/meta

```

## CLI

```bash
# run cli
rest-cmd
# show help
rest-cmd --help
```

#Quick Start

##Basic

###Concerpts
+ **Resources** and **Resource** : resource is singular(for example earth), so
 for this resource, "create", "show", "delete", "update" route map are created.
 For **Resources**, "create", "index", "delete", "update", "update" are 
 supported.
 
###Quick Example
A simplest example.

cmd_app.py

```python
from flask import Flask
from schema_sugar.contrib import FlaskJar, FlaskSugar

flask_app = Flask(__name__)
jar = FlaskJar(__name__, flask_app)

cmd = jar.entry_point

@jar.register
class SingleSugar(FlaskSugar):
    # a minimal config_dict needs field "schema", 
    # "resource" or "resources"
    config_dict = {
        "schema": {},
        # singular resource defined by "resource"
        # "create","update", "delete", "show" is supported
        "resource": "cluster",
    }
    
    # implement "show", refers to http "GET"
    def show(self, data, web_request, **kwargs):
        return {
            "field1": "hello",
        }
        
def run_server():
    app.run(debug=True)    # jar.run(debug=True) also works the same way, it's 
                           # a simple wrapper for flask app's run method
                  
```

setup.py
```
from setuptools import setup

setup(name='cmd_app',
      include_package_data=True,
      py_modules=['cmd_app'],
      entry_points={
          'console_scripts': [
              'rest-cmd = cmd_app:cmd',
              'rest-server = cmd_app:run_server',
          ],
      },
      )
```

After install, call "rest-server" to run this server.    
Now visit 
```
# resource, support GET(because only "show" method implemented)
GET host:post/cluster

# The show the document
GET host:port/cluster/meta

```

###Advanced Example
A example includes almost all features.

Refers to [cmd_app.py](examples/cmd_app.py)

##Rest API WorkFLow
When a request reaches the SchemaSugar registered view function, it
 follow given process.
```
# do what you want with data, web_request, etc
SchemaSugar.pre_precess(data, web_request, **kwargs)

# process request with given "operation", "operation" is a 
# converted string maps to method name in SchemaSugar class
# for example, "show" call SchemaSugar.show method to process data
SchemaSugar.process(operation, data, web_request, **kwargs)

# process the business logic with given opeation, for example "show"
# process_data has been validated before do real operation
SchemaSugar.given_operation(process_data, web_request, **kwargs)

# out filter
# if the operation returns a dict, out_filter will run
SchemaSugar.out_filter(result, operation)
# a lookup will be executed in config
# for out field description.

# process result with
SchemaSugar.web_response(result)
# or
SchemaSugar.cli_response(result)
# the "result" is returned by SchemaSugar.given_operation 

```

