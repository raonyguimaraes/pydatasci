![PyDataSci (wide)](logo_pds_wide.png)

---

# Value Proposition
PyDataSci's **_AIdb_** is an open source, autoML tool that keeps track of the moving parts of machine learning so that data scientists can perform best practice ML without the coding overhead.

# Mission
* **Reproducible**<br />No more black boxes. No more screenshotting parameters and loss-accuracy graphs. A record of every: dataset, feature, split, fold, parameter, run, model, and result - is persisted in a lightweight fashion. So hypertune to your heart's content, compare models, and pick the best one with the proof to back it up.<br /><br />

* **Local-first**<br />Empower non-cloud users (academic/ institute HPCs, private cloud companies, remote server SSH'ers, and everyday desktop hackers) with the same quality ML services as present in public clouds (e.g. AWS SageMaker).<br /><br />

* **Integrated**<br />Don’t disrupt the natural workflow of users by forcing them into the confines of a GUI app or specific IDE. Weave automated tracking into their existing code to work alongside the existing ecosystem of data science tools.<br /><br />

* **Scalable**<br />Queue many hypertuning jobs locally, or run big jobs in parallel in the cloud by setting `cloud_queue = True`.<br /><br />

# Painpoint Solved
In writing a paper about comparative methods for the interpretation of deep learning activation values via graph neural networks, CNNs, and LSTMs - I found myself comparing multiple models with many parameter combinations. I was burdened by questions like: Had I already tried these parameters? How was I going to save the metrics to compare the models? I was literally screenshotting my parameters and charts. That's not conducive to the scientific method. I had done the hard part in figuring out the science, but this permuted world was just a mess. When I took a look at other tools in the space, I found they were either: cloud-only, too complex/ bad documentation, incomplete (install your own database server), or too proprietary/ walled garden/ tool-biased. Let's be honest, the average data scientist isn't the world's best software engineer/ architect, so they naturally want a low-code fix for keeping track of everything.

# Functionality
- Compresses a dataset (csv, tsv, parquet) to keep track of.
- Derives informative featuresets and/ or labels from that dataset.
-- Treats validation sets (3rd split) and cross-folds (k-fold) as first-level citizens.
- Queues hypertuning jobs and batches.
- Calculates and saves performance model metrics of each model.
- Visually compares models to find the best one.
- Scales out to run cloud jobs (data size, training time) by toggling `cloud_queue = True`.

# Community
*Much to automate there is. Simple it must be.* ML is a broad space with a lot of challenges to solve. Let us know if you want to get involved. We plan to host monthly dev jam sessions and data science lightning talks. `layne <at> pydatasci.com`

* **Data types:** tabular, longitudinal, image, graph, audio, video, gaming.
* **Analysis types:** classification, regression, dimensionality reduction, feature engineering, recurrent, generative, reinforcement, NLP.

---

# Installation
Requires Python 3+. You will only need to perform these steps the first time you use the package. 

Enter the following commands one-by-one and follow any instructions returned by the command prompt to resolve errors should they arise.

_Starting from the command line:_
```bash
$ pip install --upgrade pydatasci
$ python
```
_Once inside the Python shell:_
```python
>>> import pydatasci as pds
>>> pds.create_folder()
>>> pds.create_config()
>>> from pydatasci import aidb
>>> aidb.create_db()
```

> PyDataSci makes use of the Python package, `appdirs`, for an operating system (OS) agnostic location to store configuration and database files. This not only keeps your `$HOME` directory clean, but also helps prevent careless users from deleting your database. 
>
> The installation process checks not only that the corresponding appdirs folder exists on your system but also that you have the permissions neceessary to read from and write to that location. If these conditions are not met, then you will be provided instructions during the installation about how to create the folder and/ or grant yourself the appropriate permissions. 
>
> We have attempted to support both Windows (`icacls` permissions and backslashes `C:\\`) as well as POSIX including Mac and Linux (`chmod letters` permissions and slashes `/`). Note: due to variations in the ordering of appdirs author and app directories in different OS', we do not make use of the appdirs `appauthor` directory, only the `appname` directory.
>
> If you run into trouble with the installation process on your OS, please submit a GitHub issue so that we can attempt to resolve, document, and release a fix as quickly as possible.
>
> _Installation Location Based on OS_<br />`import appdirs; appdirs.user_data_dir('pydatasci');`:
> * Mac: <br />`/Users/Username/Library/Application Support/pydatasci`<br /><br />
> * Linux - Alpine and Ubuntu: <br />`/root/.local/share/pydatasci`<br /><br />
> * Windows: <br />`C:\Users\Username\AppData\Local\pydatasci`
>
> `create_db()` is equivalent to a *migration* in Django or Rails in that it creates the tables found in the Object Relational Model (ORM). We use the [`peewee`](http://docs.peewee-orm.com/en/latest/peewee/models.html) ORM as it is simpler than SQLAlchemy, has good documentation, and found the project to be actively maintained (saw same-day GitHub response to issues on a Saturday). With the addition of Dash-Plotly, this will make for a full-stack experience that also works directly in an IDE like Jupyter or VS Code.


### Deleting & Recreating the Database
When deleting the database, you need to either reload the `aidb` module or restart the Python shell before you can attempt to recreate the database.

```python
>>> from pydatasci import aidb
>>> aidb.delete_db(True)
>>> from importlib import reload
>>> reload(aidb)
>>> create_db()
```

---

# Usage

If you've already completed the *Installation* section above, let's get started.

```python
import pydatasci as pds
from pydatasci import aidb
```

## 1. Add a `Dataset`.

Supported tabular file formats include: CSV, [TSV](https://stackoverflow.com/a/9652858/5739514), [Apache Parquet](https://parquet.apache.org/documentation/latest/). At this point, the project's support for Parquet is extremely minimal.

The bytes of the file will be stored as a BlobField in the SQLite database file. Storing the data in the database not only (a) provides an entity that we can use to keep track of experiments and link relational data to but also (b) makes the data less mutable than keeping it in the open filesystem.

```python
dataset = aidb.Dataset.create_from_file(
	path = 'iris.tsv'
	,file_format = 'tsv'
	,name = 'tab-separated plants'
	,perform_gzip = True
)
```

> You can choose whether or not you want to gzip compress the file when importing it with the `perform_gzip=bool` parameter. This compression not only enables you to store up to 90% more data on your local machine, but also helps overcome the maximum BlobField size of 2.147 GB. We handle the zipping and unzipping on the fly for you, so you don't even notice it.

### Fetch a `Dataset`.

Supported in-memory formats include: [NumPy Structured Array](https://numpy.org/doc/stable/user/basics.rec.html) and [Pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html). 

#### Pandas
```python
df = dataset.read_to_pandas()
df.head()

df2 = aidb.Dataset.read_to_pandas(id = 1)
df2.head()
```

#### NumPy
```python
arr = dataset.read_to_numpy()
arr[:4]

arr2 = aidb.Dataset.read_to_numpy(id = 1)
arr2[:4]
```
> We chose structured array because it keeps track of column names. For the sake of simplicity, we are reading into NumPy via Pandas. That way, if we want to revert to a simpler ndarray in the future, then we won't have to rewrite the function to read NumPy.

## 2. Create a `Label` if you want to perform *supervised learning* (aka predict a specific column).

From a Dataset, pick a column that you want to train against/ predict. If you are planning on training an unsupervised model, then you don't need to do this.

```python
label = aidb.Label.create_from_dataset(
	dataset_id = 1
	,column_name = 'species'
)
```

## 3. Derive a `Featureset` of columns from a Dataset.

This won't duplicate your data. It simply records the `columns` to be used in training.

#### a) For *supervised learning*, be sure to pass in the `Label` you want to predict.

```python
supervised_bruteforce = aidb.Featureset.create_all_columns(
	dataset_id = 1
	,label_id = 1
)

supervised_selective = aidb.Featureset.create_from_dataset_columns(
	dataset_id = 1
	,label_id = 1
	,columns = ['petal_width', 'petal_length']
)
```

#### b) For *unsupervised learning* (aka studying variance within a `Dataset`), leave the `Label` blank.

Feature selection is about finding out which columns in your data are most informative. In performing feature engineering, a data scientist reduces the dimensionality of the data by determining the effect each feature has on the variance of the data. This makes for simpler models in the form of faster training and reduces overfitting by making the model more generalizable to future data.

```python
unsupervised_bruteforce = aidb.Featureset.create_all_columns(
	dataset_id = 1
)

unsupervised_selective = aidb.Featureset.create_from_dataset_columns(
	dataset_id = 1
	,columns = ['petal_width', 'petal_width', 'sepal_length']
)
```


## 4. Split the `Dataset` rows into `Splitsets` based on how you want to train, test, and validate your models.

#### a) One set containing **train-test** splits.

#### b) One set containing **train-validate-test** splits.

#### c) k-fold sets containing **train-test** splits.

#### d) k-fold sets containing **train-validate-test** splits.


## 5. Create an `Algorithm` aka model to fit to your splits.


## 6. Create combinations of `Hyperparamsets` for your algorithms.


## 7. Create a `Batch` of `Job`'s to keep track of training.

---

# PyPI Package

### Steps to Build & Upload

```bash
$ pyenv activate pydatasci
$ pip3 install --upgrade wheel twine
$ python3 setup.py sdist bdist_wheel
$ python3 -m twine upload --repository pypi dist/*
$ rm -r build dist pydatasci.egg-info
# proactively update the version number in setup.py next time
$ pip install --upgrade pydatasci; pip install --upgrade pydatasci
```