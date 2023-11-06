# Fabric
The following code snippets may be executed in a [Microsoft Fabric notebook](https://learn.microsoft.com/fabric/data-engineering/how-to-use-notebook). These scripts leverage the Semantic-Link (a.k.a. Sempy) library.

#### Load the [Semantic Link](https://pypi.org/project/semantic-link/) library inside of your notebook
```python
%pip install semantic-link
```

#### Useful packages for python notebooks (Sempy and more)
```python
import sempy
import sempy.fabric as fabric
import pandas as pd
import pyspark.sql.functions as F
```

### Show the directory of all sempy functions
```python
import sempy.fabric
dir(sempy.fabric)
```

#### Show useful information about a given Sempy function
```python
import sempy.fabric as fabric
help(fabric.list_datasets) # replace 'list_datasets' within any function shown in the dir(sempy.fabric) output
```

## Workspace/Lakehouse objects

#### Gets the Lakehouse ID from the current lakehouse
```python
import sempy.fabric as fabric
x = fabric.get_lakehouse_id()
x
```

#### Gets the Artifact ID (of the notebook)
```python
import sempy.fabric as fabric
x = fabric.get_artifact_id()
x
```

#### Shows a list of your accessible workspaces, sorted alphabetically
```python
import sempy.fabric as fabric
df = fabric._list_workspaces().sort_values(by='Name')
df
```

#### Filter to a particular workspace
```python
import sempy.fabric as fabric
workspaceName = "MK Demo 4" #Enter the workspace name to be used as a filter
df = fabric._list_workspaces()
filter_condition = [workspaceName]
df = df[df['Name'].isin(filter_condition)]
df
```

#### Find the workspace ID for a given workspace name
```python
import sempy.fabric as fabric
workspaceID = fabric.resolve_workspace_id(workspaceName)
workspaceID
```

#### Find the workspace name for a given workspace ID
```python
import sempy.fabric as fabric
workspaceID = "" #Enter the workspace ID
workspaceName = fabric.resolve_workspace_name(workspaceID)
workspaceName
```

#### Get the current workspace ID
```python
import sempy.fabric as fabric
workspaceID = fabric.get_workspace_id()
workspaceID
```

## Dataset and dataset objects

#### Shows a list of datasets in your current workspace
```python
import sempy.fabric as fabric
x = fabric.list_datasets()
x
```

#### Shows the [TMSL](https://learn.microsoft.com/analysis-services/tmsl/tabular-model-scripting-language-tmsl-reference?view=asallproducts-allversions) for a given dataset
```python
import sempy.fabric as fabric
datasetName = "" #Enter your dataset name
workspaceName = "" #Enter your workspace name
x = fabric.get_tmsl(datasetName, workspaceName)
print(x)
```

#### List the tables within a given dataset (semantic model)
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
df = fabric.list_tables(datasetName)
df
```

#### List the columns within a given dataset (semantic model)
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
df = fabric.list_tables(datasetName, True)
df
```

#### List the partitions within a given dataset (semantic model)
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
df = fabric.list_tables(datasetName, False, True)
df
```

#### List the measures within a given dataset (semantic model)
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
df = fabric.list_measures(datasetName)
df
```

#### List the relationships within a given dataset (semantic model)
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
df = fabric.list_relationships(datasetName)
df
```

#### Plot the relationships for a given dataset (semantic model)
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
relationships = fabric.list_relationships(datasetName)
plot_relationship_metadata(relationships)
```

## Tabular Object Model

#### Connect to the [Tabular Object Model](https://learn.microsoft.com/analysis-services/tom/introduction-to-the-tabular-object-model-tom-in-analysis-services-amo?view=asallproducts-allversions) ([TOM](https://learn.microsoft.com/dotnet/api/microsoft.analysisservices.tabular.model?view=analysisservices-dotnet)); prints each table name
```python
import sempy.fabric as fabric
from sempy.fabric._client import DatasetXmlaClient
from sempy.fabric._cache import _get_or_create_workspace_client
sempy.fabric._client._utils._init_analysis_services()
workspaceName = "" #Enter workspace name
datasetName = "" #Enter dataset name
workspace_client = _get_or_create_workspace_client(workspaceName)
ds = workspace_client.get_dataset(datasetName)

m = ds.Model

for t in m.Tables:
	print(t.Name)
```

## DAX

#### Run DAX via evaluate_dax()
```python
import sempy.fabric as fabric
datasetName = "" #Enter dataset name
df = fabric.evaluate_dax(
    datasetName,
    """
    EVALUATE
    SUMMARIZECOLUMNS(
    "Header Name",[Measure Name]
    )
    """
    )
df
```

#### Run [Dynamic Management Views](https://learn.microsoft.com/analysis-services/instances/use-dynamic-management-views-dmvs-to-monitor-analysis-services?view=asallproducts-allversions) (DMVs) via evaluate_dax()
```python
import sempy.fabric as fabric
datasetName = ""
df = fabric.evaluate_dax(
        datasetName,
        """
        SELECT
        MEASURE_GROUP_NAME AS [TableName]
        ,ATTRIBUTE_NAME AS [ColumnName]
        ,DATATYPE AS [DataType]
        ,DICTIONARY_SIZE AS [DictionarySize]
        ,DICTIONARY_ISRESIDENT AS [IsResident]
        ,DICTIONARY_TEMPERATURE AS [Temperature]
        ,DICTIONARY_LAST_ACCESSED AS [LastAccessed]
        FROM $SYSTEM.DISCOVER_STORAGE_TABLE_COLUMNS
        WHERE [COLUMN_TYPE] = 'BASIC_DATA'
        AND NOT [ISROWNUMBER]
        ORDER BY [DICTIONARY_TEMPERATURE] DESC
        """)
df
```

#### Run a single measure against 1+ columns in your dataset
```python
import sempy.fabric as fabric
df = fabric.evaluate_measure(
    "DatasetName", #Enter your dataset name
    "MeasureName", #Enter the measure name from your dataset   
    ["'TableName'[ColumnName]", "TableName[ColumnName]"]) # Enter columns
df
```

#### Enable DAX cell magic (ability to run DAX directly in a notebook cell using %%dax)
```python
%load_ext sempy
```

#### Run DAX using DAX cell magic
```python
%%dax "DatasetName" -w "WorkspaceName" # Enter DatasetName & WorkspaceName 

EVALUATE
SUMMARIZECOLUMNS(
"Header Name",[Measure Name]
)
```

#### Run [Dynamic Management Views](https://learn.microsoft.com/analysis-services/instances/use-dynamic-management-views-dmvs-to-monitor-analysis-services?view=asallproducts-allversions) (DMVs) using DAX cell magic
```python
%%dax "DatasetName" -w "WorkspaceName" # Enter DatasetName & WorkspaceName 

SELECT * FROM $SYSTEM.DISCOVER_SESSIONS
```
