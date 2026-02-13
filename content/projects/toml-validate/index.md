---
title: "TOML Validation"
date: 2026-02-13
description: "TOML Validation"
summary: "A config validator targeting TOML files."
tags: ["python"]
draft: false
---

# Introduction

This was an idea I initially worked on at Kasalis. I rewrote it into an open-source project.

{{< github repo="ejbosia/toml-validate" showThumbnail=false >}}

> _In hindsight, I would have used Pydantic rather than writing my own validator. It would have saved a lot of time. See [Pydantic Retrospective](#pydantic-retrospective)._

# Background

Data validation is important to avoid runtime errors. This is especially important in Python, where any variables can hold any type (thanks duck typing!). It is better to fail on config load rather than during a process.

The Kasalis machines used [TOML](https://toml.io/en/) configuration files.

**Stakeholders**
 - **Process Engineers**: non-developers who set up the process of the machine. They interacted the most with the TOML config files.

**Goals**
 - Define the schema in a readable format.
 - Validate the **structure** and **contents** of the config.
 - Report all failures in a readable format.

# Design

My goal was to make TOML validation easy to use and customizable. To do this, I broke the process into four steps, where each step requires the last:

1. Define the rules available for validation.
1. Build the schema.
1. Validate the data against the schema.
1. Create a report of the validation.

Most of this logic is contained in the `TomlValidator` class. I chose to make this a class so that the available rules and schema could be state. The schema should only need to be built once but the validation could be run many times.

## Custom Rules

I decided to separate defining the available rules from defining the schema. Rules require logic which is easier to write in Python. The schema could then be written in TOML, because it would just define what rule to run and what data to compare.

Defining a rule maps a keyword to a function. I decided to make rules always expect a comparison value to better support the `key=value` structure of TOML tables. The key would be the rule, and the value would be the comparison value.

I added the `ValidationFunction` Protocol to support this. This allowed for custom validation functions, so long as they matched this Protocol.

```python
class ValidationFunction(Protocol):
    """
    Protocol representing a function that validates a value.
    """
    def __call__(self, val: Any, comp: Any) -> TomlError: ...
```

The Protocol enforces the `val` and `comp` parameters as well as the `TomlError` return value when adding a new validation rule. This allows the build process to safely assign the `comp` value from the TOML key-value pair. For example, these would be all of the components for defining an "equals" rule:

```python
from toml_validate import *

def check_equals(val, comp) -> TomlError:
    if val == comp:
        return TomlError(ok=True, message=f"{val} is equal to {comp}")
    else:
        return TomlError(ok=False, message=f"{val} is not equal to {comp}")
    

validator = TomlValidator()

# Map the "_equals" keyword to the check_equals function.
validator.add_validation_rule("_equals", check_equals)
```

Then in the schema, the new rule can be used.

```toml
# foo must an int equal to 1
foo = {_type="int", _equals=1}
```

## Templates

Templates allow for reusable schema definitions. I added an `add_template` method. During the schema build, strings detected are replaced with templates if possible. This improves readability of the schema.

```python
from toml_validate import TomlValidator

validator = TomlValidator.default_validator()
# Add a custom template for percents.
validator.add_template("percent", {"_type": "float", "_min": 0.0, "_max": 100.0})
```

The schema can now use the `percent` template.

```toml
# Without templates
a = {_type="float", _min=0.0, _max=100.0}
b = {_type="float", _min=0.0, _max=100.0}

# With templates
c = "percent"
d = "percent"
```

The `default_validator` function creates a `TomlValidator` with type templates. That way, it is possible to define an integer with `"int"` rather than `{_type="int"}` for example.

## Schema Design

The goal of this project was to validate the **structure** and **contents** of a TOML file.

 - **structure**: The expected sections and keys are present.
 - **contents**: The values are of expected types and pass other validation rules.

I decided to make the structure of the schema file match the expected structure of the data file. This made it obvious what structure the data should be based on the schema.

The validation rules were defined in a inline table. This allowed for the definition of multiple rules for a single value. This also preserves the readability of the structure.

```toml
# Structure: the key foo.bar.x is present.
# Contents:  the key foo.bar.x is an int that is minimum 0 and maximum 10.
[foo.bar]
    x = {_type="int", _min=0, _max=10}
```

The `_type` keyword was used to differentiate between sections and keys. The `_type` is set to `dict` if the key contains dictionary data and `_type` is not present. I decided to use the python style for private values because `type` could be a config value, which would create a collision case.

```toml
dict_expected = {}           # _type="dict" is added by default
int_expected = {_type="int"} # _type="int" means this is an integer
```

Lists are special. In order to support TOML, I made the expected schema for a list contain a single dictionary value defining both the list and item validation. For simple value lists, I added a special `_schema` key.

```toml
# Both examples rely on the templates from TomlValidate.default_validator().

# Create a list of ints between length 1 and 3.
int_list = [{_min_length=1, _max_length=3, _schema="int"}]

# Create a list of dicts between length 1 and 3.
[[section_list]]
    _min_length = 1
    _max_length = 3
    a = "int"
    b = "float"
    c = "str"
```

## Validation Design

TOML files contain a nested dict/list. Traversal of this data format can be done using recursion.

The schema is the ground truth, so it is traversed. The validation result matches this structure with recursive dictionaries, where the terminal leaves are `TomlError` objects. List elements are keyed on the index. Each `ValidationFunction` returns a `TomlError` object. This contains both the result of the validation and a message. This allows for printing both passing and failing values.

For example, this schema...
```toml
[foo]
    bar = "int"
    baz = {_type: "int", _options=[1,2,3]}
```

run against this data...

```toml
[foo]
            # bar is missing
    baz = 5 # baz is int, but is not in [1,2,3] 
```

produces this result structure...

```python
{
    "foo": {
        "bar": TomlError(ok=False, message="bar not present in data"),
        "baz": {
            "_type": TomlError(ok=True, message="type of 5, \"int\", matches type \"int\""),
            "_options": TomlError(ok=False, message="5 not in [1,2,3]"),
        }
    }
}
```

which outputs this report...

```
[foo]
    bar: FAIL - bar not preset in data
[foo.baz]
    _options: FAIL - 5 not in [1,2,3]
```

The `TomlValidationResult` contains this result structure. It lazily determines the success of the entire validation with the `ok` property. It can also output a report that shows the result for each key, either failures only or all validation results.

# Pydantic Retrospective

If I had a time machine, I would have my past self use [Pydantic](https://docs.pydantic.dev/latest). When I researched existing validators, Pydantic came up. I decided against using it because:

 - The syntax seemed too complicated for the process engineers.
 - I was focused on TOML files only. Pydantic requires python object representation.
 - It's more fun to write software, right? Right??? **RIGHT???**

The syntax concerns are not as important as I thought. LLMs help a lot with the syntax. TOML files are converted to python types so that is not an issue.

In retrospect, Pydantic has some huge advantages over my implementation:

 - **IDE Support**: The object representation of the data allows for auto-complete in the IDE. A common runtime error I encountered were typos when _accessing_ configs.
 - **Cross-field Validation**: Fields that have relationships can be validated. For example, an axis position must be between the min and max limits.
 - **Development Time**: I spent more time building the validation tool than if I implemented validation with Pydantic.
 - **Reliability**: Pydantic has been used by thousands of projects and some major tech companies. It is battle tested. That is not true about my implementation.

# Conclusion

This was a fun, pure-python project. It was also a good lesson in using existing functionality when possible.
