[Back to Cookbook Menu](../..)

# Types in Flyte through Flytekit (python)


Flyte's comes with its own type system, which closely maps most programming languages. These types are what power Flyte's magic of
 - Data lineage
 - Memoization
 - Auto parallelization
 - Simplifying access to data
 - Auto generated CLI and Launch UI

## Mapping of Types between Python and Flyte
As of flytekit 0.10.0 (notice why I added 0.10.0, we are working hard to auto-magically making it work for you - but until then), Users need to annotated their tasks and workflows and map their inputs and outputs to Flyte's own types. The following tables help you create a mental model of how to map the types. Once we understand the mapping, later sections will point to examples of how to use these types.

```python
from flytekit.sdk.types import Types
```

### Primitive types
| Python          | Flyte         |
|-----------------|---------------|
| int             | Types.Integer |
| float           | Types.Float   |
| str             | Types.String  |
| bool            | Types.Boolean |
| bytes/bytearray | Not supported*| 
| complex         | Not supported |
 
 *we have binary in flyte, but flytekit today does not support it

### Time types
| Python             | Flyte           |
|--------------------|-----------------|
| datetime.timedelta | Types.Timedelta |
| datetime.datetime  | Types.Datetime  |

### Container types
| Python            | Flyte          |
|-------------------|----------------|
| List              | Types.List     |

Flyte type engine supports Dictionary type, but today it is not exposed to users. This is because the key can only be a string.

### IO Types
| Python                              | Flyte                 |
|-------------------------------------|-----------------------|
| file/file-like                      | Types.Blob, Types.Csv |
| Directory like                      | Types.MultipartBlob   |
| Pandas Dataframe (stored as a file) | Types.Schema*         |

**Types.Schema** Can represent any structured columnar or row structure. For example it could be used for Spark dataframes, vaex dataframes, raw parquet files, RecordIO structure etc

### Other types
| Python          | Flyte         |
|-----------------|---------------|
| JSON, json-like | Types.Generic |
| Proto           | Types.Proto*  |

**Types.Proto** is essentially passed through a binary byte array structure.

## Use primitives 
[Example: primitive.py](primitive.py)

Primitives are directly passed as Python standard types. Refer to the table for the conversion. 
To Return a primitive you have to use the set method on the passed in reference. 

```bash
 flyte-cli -h localhost:30081 -i list-launch-plan-versions -p flytesnacks -d development | grep PrimitiveDemoWorkflow

 flyte-cli -h localhost:30081 -i execute-launch-plan -p flytesnacks -d development -u <urn> -r kumare -- x=10 y=10.0 s="Hello" b=True
```

## Use Time types

[Example: time.py](time.py)

The CLI accepts *datetime* and *duration* fields in [RFC3339](https://tools.ietf.org/html/rfc3339 ) formats, which is usually of the form **YYYYMMDDTHH:MM:SSZ** (z -> timezone). Duration is of the
format **10H** (for 10 hours) or **10S** or **2D** (days etc)

```bash
  # To retrieve the right LaunchPlan Urn:
  flyte-cli -h localhost:30081 -i list-launch-plan-versions -p flytesnacks -d development | grep TimeDemoWorkflow
  # Then take the URN and plug here
  flyte-cli -h localhost:30081 -i execute-launch-plan -p flytesnacks -d development -u <urn> -r kumare -- dt=20200707T00:00Z duration=10H
```
*Coming soon*

## Use container types
*Coming soon*

## Use Blob

## Use CSV

## Use MultipartBlob

## Use Schema and pandas data frames

## Use / pass - Json / Json like / Custom Python structs

[Example: generic.py](generic.py)

One of the types that Flyte supports is a `struct type <https://github.com/lyft/flyteidl/blob/f8181796dc5cafe019b1493af1b64384ae1358f5/protos/flyteidl/core/types.proto#L20>`__.  The addition of this type was debated internally for some time as it effectively removes type-checking from Flyte, so use this with caution. It is a `scalar literal <https://github.com/lyft/flyteidl/blob/f8181796dc5cafe019b1493af1b64384ae1358f5/protos/flyteidl/core/literals.proto#L63>`__, even though in most cases it won't represent a scalar.

Flytekit implements this `here <https://github.com/lyft/flytekit/blob/1926b1285591ae941d7fc9bd4c2e4391c5c1b21b/flytekit/common/types/primitives.py#L501>`__.  

The UI currently does not support passing structs as inputs to workflows, so if you need to rely on this, you'll have to use **flyte-cli** for now. ::

```bash
  flyte-cli -p flytesnacks -d development execute-launch-plan -u lp:flytesnacks:development:recipes.types.generic.GenericDemoWorkflow:version -r demo -- a='{"a": "hello", "b": "how are you", "c": ["array"], "d": {"nested": "value"}}'
```

## Pass your own proto objects
