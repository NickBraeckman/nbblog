---
title: "Azure Machine Learning: Exploring Data Concepts"
datePublished: Fri Oct 27 2023 08:42:37 GMT+0000 (Coordinated Universal Time)
cuid: clo8d7y7v000l09jv86uh1xa2
slug: azure-machine-learning-exploring-data-concepts
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1698395763432/2761d97f-7fde-47c5-a7db-0b6f0b72f7a0.png
tags: data-science, data-management, azureml, azure-machine-learning, data-assets

---

[Azure Machine Learning](https://learn.microsoft.com/en-us/azure/machine-learning/?view=azureml-api-2) allows you to import data from your local machine or existing cloud-based storage resource into your workspace.

In this post, we'll take a practical look at importing various data types, such as **File**, **Folder** and **Table** types. We'll use the [Azure Machine Learning Python SDK v2](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-ml-readme?view=azure-python).

## Prerequisites

- An Azure Subscription:  If you don't have one, don't worry. You can create a [free Azure account](https://azure.microsoft.com/free/) with a limited $200 USD credit to get started.
- An [Azure Machine Learning Workspace](https://learn.microsoft.com/en-us/azure/machine-learning/quickstart-create-resources?view=azureml-api-2): If you haven't set up your workspace yet, you can follow the quick start guide to create the necessary resources.
- The [Azure Machine Learning Python SDK v2](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-ml-readme?view=azure-python&viewFallbackFrom=azure-ml-py): To leverage the power of Azure Machine Learning, ensure that you have the Python SDK v2 installed.

## Setting up Your Environment

To start importing data into your Azure Machine Learning workspace, you'll need to prepare your environment. We'll be using the `mltable` package for data loading, so make sure you install it in your Python environment. Run the following command to install the necessary packages:

```bash
pip install mltable azureml-dataprep[pandas]
```

## Understanding Data Types and URIs

Azure Machine Learning allows you to work with three primary data types - Files, Folders, and Tables. Each data type serves a specific purpose, making it crucial to understand when and how to use them.

- **File**: read/write a single file, for instance storing a `.pkl` file of your serialized model on your local environment after training.
- **Folder**: read/write a folder of files, for instance importing a folder with images for training a deep-learning model.
- **Table**: read/write via a dynamic schema, for instance you want to start an [AutoML](https://learn.microsoft.com/en-us/azure/machine-learning/concept-automated-ml?view=azureml-api-2) job.

Each data type can be referenced by a Uniform Resource Identifier (URI). The URI represents the storage location on your local computer or on an external resource. There are different URIs for each storage location. Here are some URI examples:

- Azure Machine Learning Datastore: `azureml://datastores/<data_store_name>/paths/<folder1>/<folder2>//<file>.csv`
- Local computer: `./home/user/data/my_data`
- Public http(s) server: `https://raw.githubusercontent.com/pandas-dev/pandas/main/doc/data/titanic.csv`
- Blob storage: `wasbs://<containername>@<accountname>.blob.core.windows.net/<folder>/`
- Azure Data Lake Gen1: `abfss://<file_system>@<account_name>.dfs.core.windows.net/<folder>/<file>.csv`
- Azure Data Lake Gen2: `adl://<accountname>.azuredatalakestore.net/<folder1>`

Instead of using long storage paths of URIs, we'll create data assets, whereby we can access the data via a friendly name. The data asset is, just like the URI, a reference to the data source location, along with a copy of its metadata.

## Creating Data Assets

Project structure:

``` 
project/
     |-- data/
     |-- mltable/
     |-- scripts/
     |-- config.json
```

### Step 1: Setting up the MLClient

In this step, we'll instantiate the MLClient using a `config.json` file. Make sure you have the necessary configuration file in place, as shown below:

```json
{
    "subscription_id": "<YOUR_SUBSCRIPTION_ID>",
    "resource_group": "<YOUR_RESOURCE_GROUP>",
    "workspace_name": "<YOUR_WORKSPACE_NAME>"
}
```

You can test your configuration by executing the following code in the Python interpreter in [interactive mode](https://docs.python.org/3/tutorial/interpreter.html#interactive-mode).

```python
>>> from azure.ai.ml import MLClient
>>> from azure.identity import DefaultAzureCredential
>>> ml_client = MLClient.from_config(DefaultAzureCredential())
>>> print(ml_client)
```

### Step 2: Add Data

Add a file to the `.\data` directory. You can use the [this titanic example dataset](https://raw.githubusercontent.com/pandas-dev/pandas/main/doc/data/titanic.csv) if you don't have any file.

### Step 3: Create a File data asset

Navigate to the `. \script` folder and create a Python file named `create_file_data_asset.py`. This script will register the file as a data asset.

```python
# create_file_data_asset.py
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

if __name__ == '__main__':

    ml_client = MLClient.from_config(DefaultAzureCredential())

    version = '<VERSION>'
    path = './data/<FILE_NAME>'
    name = '<DATA_ASSET_NAME>'
    description = '<DATA_ASSET_DESCRIPTION>'

    data_asset = Data(
        path=path,
        type=AssetTypes.URI_FILE,
        description=description,
        name=name,
        version=version
    )
    try:
        ml_client.data.create_or_update(data_asset)
        print(f'Dataset `{name}` version `{version}` created/updated successfully.')
    except Exception as e:
        print(f'Error: {str(e)}')
```
### Step 4: Create a Table data asset

In the previous step, we registered the file as a File data asset. However, if your data is tabular, you can create an MLTable from it and register it as a data asset. An MLTable is a schema that defines a set of operations for loading data from the source. We'll author an MLTable file using `mltable`, which will document the data loading operations, and create an MLTable data asset.

```python
# to_ml_table.py
import mltable
from mltable import MLTableHeaders, MLTableFileEncoding
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential


if __name__ == '__main__':

    ml_client = MLClient.from_config(DefaultAzureCredential())

    version = '<FILE DATA ASSET VERSION>'
    name = '<FILE DATA ASSET NAME>'

    data_asset = ml_client.data.get(name=name, version=version)

    tbl = mltable.from_delimited_files(
        paths=[{'file': data_asset.path}],
        delimiter=',',
        header=MLTableHeaders.all_files_same_headers,
        infer_column_types=True,
        include_path_column=False,
        encoding=MLTableFileEncoding.utf8
    )

    print(f'MLTable `{name}` version `{version}` created from delimted files successfully.')

    print(tbl.show(5))
    df = tbl.to_pandas_dataframe()
    print(df.head(5))
    tbl.save('./mltable/<MLTABLE NAME>')
```

You can take a look in the `.\mltable\<MLTABLE NAME>` folder to find a file called `MLTable`. This is a `YAML` file that describes the operations for data loading. 

```yaml
paths:
- file: azureml://subscriptions/<subscription_id>/resourcegroups/<resource_group_name>/workspaces/<workspace_name>/datastores/<datastore_name>/paths/LocalUpload/<path_id>/<file_name>
transformations:
- read_delimited:
    delimiter: ','
    empty_as_string: false
    encoding: utf8
    header: all_files_same_headers
    include_path_column: false
    infer_column_types: true
    partition_size: 20971520
    path_column: Path
    support_multi_line: false
type: mltable
```

You can load the MLTable as follows:

```python
# load_ml_table.py
import mltable

if __name__ == '__main__':

    tbl = mltable.load('./mltable/<MLTABLE NAME>')
    print(tbl.show(5))
```

Next we can create a Table data asset using the following code:

```python
# create_ml_table.py
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

if __name__ == '__main__':

    ml_client = MLClient.from_config(DefaultAzureCredential())

    path = './mltable/<MLTABLE NAME>'
    version = '<MLTABLE DATA ASSET VERSION>'
    name = '<MLTABLE DATA ASSET NAME>'
    description = '<MLTABLE DATA ASSET DESCRIPTION>'

    data_asset = Data(
        path=path,
        type=AssetTypes.MLTABLE,
        description=description,
        name=name,
        version=version
    )

    try:
        ml_client.data.create_or_update(data_asset)
        print(f'MLTable `{name}` version `{version}` created/updated successfully.')
    except Exception as e:
        print(f'Error: {str(e)}')
```

### Step 5: Archive a data asset

[Azure Machine Learning](https://learn.microsoft.com/en-us/azure/machine-learning/?view=azureml-api-2) does not support data asset deletion due to its [immutable](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-create-data-assets?view=azureml-api-2&tabs=cli#delete-a-data-asset) nature. Instead, you can archive the data asset. Use the provided Python script to archive a previously created File Data Asset.

```python
# archive_data_asset.py
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

if __name__ == '__main__':

    ml_client = MLClient(DefaultAzureCredential())

    name = '<DATA ASSET NAME>'

    ml_client.data.archive(name)

```

---

In this post, we've only scratched the surface of what [Azure Machine Learning](https://learn.microsoft.com/en-us/azure/machine-learning/?view=azureml-api-2) can do for your data management needs. There are many more exciting features and capabilities waiting to be explored. Stay tuned for more in-depth guides and tutorials on Azure Machine Learning, where we'll dive deeper into its functionalities.







 

 


