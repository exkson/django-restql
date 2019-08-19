# django-restql

[![Latest Version](https://img.shields.io/pypi/v/django-restql.svg)](https://pypi.org/project/django-restql/)
[![Python Versions](https://img.shields.io/pypi/pyversions/django-restql.svg)](https://pypi.org/project/django-restql/)
[![License](https://img.shields.io/pypi/l/django-restql.svg)](https://pypi.org/project/django-restql/)

**django-restql** is a python library which allows you to turn your API made with **Django REST Framework(DRF)** into a GraphQL like API. With this you will be able to
* Send a query to your API and get exactly what you need, nothing more and nothing less.

* Control the data you get, not the server.

* Get predictable results, since you control what you get from the server.

* Save the load of fetching unused data from the server.

Isn't it cool?.


## Installing

```python
pip install django-restql
```

## Getting Started
Using **django-restql** is very simple, you just have to inherit the `DynamicFieldsMixin` class when defining a serializer.
```python
from rest_framework import serializers
from django.contrib.auth.models import User

from django_restql import DynamicFieldsMixin

class UserSerializer(DynamicFieldsMixin, serializer.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'groups']
```

A regular request returns all fields as specified on DRF serializer, in fact **django-restql** doesn't handle this request at all:

`GET /users`

``` json
    [
      {
        "id": 1,
        "username": "yezyilomo",
        "email": "yezileliilomo@hotmail.com",
        "groups": [1,2]
      },
      ...
    ]
```

**django-restql** handle all GET requests with `query` parameter, this parameter is the one used to pass all fields to be included in a response. For example to select `id` and `username` fields from `user` model, send a request with a ` query` parameter as shown below.

`GET /users/?query={id, username}`

```json
    [
      {
        "id": 1,
        "username": "yezyilomo"
      },
      ...
    ]
```

**django-restql** support querying both flat and nested resources, so you can expand or query nested fields at any level as long as your field is defined as nested field on a serializer. For example you can query a country and region field from location.

`GET /users/?query={id, username, location{country, region}}`

```json
    [
      {
        "id": 1,
        "username": "yezyilomo",
        "location": {
            "contry": "Tanzania",
            "region": "Dar es salaam"
        }
      },
      ...
    ]
```

**django-restql** got your back on querying iterable nested fields(one2many or many2many) too. For example if you want to expand `groups` field into `id` and `name`, here is how you would do it.

`GET /users/?query={id, username, groups{id, name}}`

```json
    [
      {
        "id": 1,
        "username": "yezyilomo",
        "groups": [
            {
                "id": 2,
                "name": "Auth_User"
            }
            {
                "id": 3,
                "name": "Admin_User"
            }
        ]
      },
      ...
    ]
```

If a query contains nested field without expanding and it's not defined as a nested field on a serializer, **django-restql** will return its id or array of ids for the case of nested iterable field(one2many or many2many). For example on a request below `location` is a flat nested field(many2one) and `groups` is an iterable nested field(one2many or many2many).

`GET /users/?query={id, username, location, group}`

```json
    [
      {
        "id": 1,
        "username": "yezyilomo",
        "location": 6,
        "groups": [1,2]
      },
      ...
    ]
```


## Customizing django-restql
**django-restql**  is very configurable, here is what you can customize on it.
* Change the name of ```query``` parameter.

    If you don't want to use the name ```query``` as your parameter, you can inherit `DynamicFieldsMixin` and change it as shown below
    ```python
    from django_restql.mixins import DynamicFieldsMixin

    class MyDynamicFieldMixin(DynamicFieldsMixin):
        query_param_name = "your_favourite_name"
     ```
     Now you can use this Mixin on your serializer and use the name `your_favourite_name` as your parameter. E.g

     `GET /users/?your_favourite_name={id, username}`

* Customize how fields to include in a response are filtered.
    You can do this by inheriting DynamicFieldsMixin and override `field` methods as shown below.

    ```python
    from django_restql.mixins import DynamicFieldsMixin

    class CustomDynamicFieldMixin(DynamicFieldsMixin):
        @property
        def fields(self):
            # Your customization here
            return fields
    ```
    **Note:** To be able to do this you must understand how **django-restql** is implemented, specifically **DynamicFieldsMixin** class, you can check it [here](https://github.com/yezyilomo/django-restql/blob/master/django_restql/mixins.py). In fact this is how **django-restql** is implemented(just by overriding `field` method of a serializer, nothing more and nothing less).


## Running Tests
`python setup.py test`


## Credits
* Implementation of this library is based on the idea behind [GraphQL](https://graphql.org/).
* The most important part of this library which does the filtering work is powered by [dictfier](https://github.com/yezyilomo/dictfier) library.
* My intention is to extend the capability of [drf-dynamic-fields](https://github.com/dbrgn/drf-dynamic-fields) library to support more functionalities like allowing to query nested fields both flat and iterable at any level, while maintaining simplicity.


## Contributing [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

We welcome all contributions. Please read our [CONTRIBUTING.md](https://github.com/yezyilomo/django-restql/blob/master/CONTRIBUTING.md) first. You can submit any ideas as [pull requests](https://github.com/yezyilomo/django-restql/pulls) or as [GitHub issues](https://github.com/yezyilomo/django-restql/issues). If you'd like to improve code, check out the [Code Style Guide](https://github.com/yezyilomo/django-restql/blob/master/CONTRIBUTING.md#styleguides) and have a good time!.
