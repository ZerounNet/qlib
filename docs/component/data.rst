.. _data:
================================
Data Layer: Data Framework&Usage
================================

Introduction
============================

``Data Layer`` provides user-friendly APIs to manage and retrieve data. It provides high-performance data infrastructure. 

It is designed for quantitative investment. For example, users could build formulaic alphas with ``Data Layer`` easily. Please refer to `Building Formulaic Alphas <../advanced/alpha.html>`_ for more details.

The introduction of ``Data Layer`` includes the following parts.

- Data Preparation
- Data API
- Data Handler
- Cache
- Data and Cache File Structure


Data Preparation
============================

Qlib Format Data
------------------

We've specially designed a data structure to manage financial data, please refer to the `File storage design section in Qlib paper <https://arxiv.org/abs/2009.11189>`_ for detailed information.
Such data will be stored with filename suffix `.bin` (We'll call them `.bin` file, `.bin` format or qlib format). `.bin` file is designed for scientific computing on finance data

Qlib Format Dataset
--------------------
``Qlib`` has provided an off-the-shelf dataset in `.bin` format, users could use the script ``scripts/get_data.py`` to download the dataset as follows.

.. code-block:: bash

    python scripts/get_data.py qlib_data_cn --target_dir ~/.qlib/qlib_data/cn_data

After running the above command, users can find china-stock data in Qlib format in the ``~/.qlib/csv_data/cn_data`` directory.

``Qlib`` also provides the scripts in ``scripts/data_collector`` to help users crawl the latest data on the Internet and convert it to qlib format.

When ``Qlib`` is initialized with this dataset, users could build and evaluate their own models with it.  Please refer to `Initialization <../start/initialization.html>`_ for more details.

Converting CSV Format into Qlib Format
-------------------------------------------

``Qlib`` has provided the script ``scripts/dump_bin.py`` to convert data in CSV format into `.bin` files(Qlib format).


Users can download the china-stock data in CSV format as follows for reference to the CSV format.

.. code-block:: bash

    python scripts/get_data.py csv_data_cn --target_dir ~/.qlib/csv_data/cn_data


Supposed that users prepare their CSV format data in the directory ``~/.qlib/csv_data/my_data``, they can run the following command to start the conversion.

.. code-block:: bash

    python scripts/dump_bin.py dump --csv_path  ~/.qlib/csv_data/my_data --qlib_dir ~/.qlib/qlib_data/my_data --include_fields open,close,high,low,volume,factor

After conversion, users can find their Qlib format data in the directory `~/.qlib/qlib_data/my_data`.

.. note::

    The arguments of `--include_fields` should correspond with the columns names of CSV files. The columns names of dataset provided by ``Qlib`` includes open,close,high,low,volume,factor.
    
    - `open`
        The opening price
    - `close`
        The closing price
    - `high`
        The highest price
    - `low`
        The lowest price
    - `volume`
        The trading volume
    - `factor`
        The Restoration factor


China-Stock Mode & US-Stock Mode
--------------------------------

- If users use ``Qlib`` in china-stock mode, china-stock data is required. Users can use ``Qlib`` in china-stock mode according to the following steps:
    - Download china-stock in qlib format, please refer to section `Qlib Format Dataset <#qlib-format-dataset>`_.
    - Initialize ``Qlib`` in china-stock mode
        Supposed that users download their Qlib format data in the directory ``~/.qlib/csv_data/cn_data``. Users only need to initialize ``Qlib`` as follows.
        
        .. code-block:: python

            from qlib.config import REG_CN
            qlib.init(provider_uri='~/.qlib/qlib_data/cn_data', region=REG_CN)
        

- If users use ``Qlib`` in US-stock mode, US-stock data is required. ``Qlib`` does not provide a script to download US-stock data. Users can use ``Qlib`` in US-stock mode according to the following steps:
    - Prepare data in CSV format
    - Convert data from CSV format to Qlib format,  please refer to section `Converting CSV Format into Qlib Format <#converting-csv-format-into-qlib-format>`_.
    - Initialize ``Qlib`` in US-stock mode
        Supposed that users prepare their Qlib format data in the directory ``~/.qlib/csv_data/us_data``. Users only need to initialize ``Qlib`` as follows.
        
        .. code-block:: python

            from qlib.config import REG_US
            qlib.init(provider_uri='~/.qlib/qlib_data/us_data', region=REG_US)
        

Data API
========================

Data Retrieval
---------------
Users can use APIs in ``qlib.data`` to retrieve data, please refer to `Data Retrieval <../start/getdata.html>`_.

Feature
------------------

``Qlib`` provides `Feature` and `ExpressionOps` to fetch the features according to users' needs.

- `Feature`
    Load data from the data provider. User can get the features like `$high`, `$low`, `$open`, `$close`, .etc, which should correspond with the arguments of `--include_fields`, please refer to section `Converting CSV Format into Qlib Format <#converting-csv-format-into-qlib-format>`_.

- `ExpressionOps`
    `ExpressionOps` will use operator for feature construction.
    To know more about  ``Operator``, please refer to `Operator API <../reference/api.html#module-qlib.data.ops>`_.

To know more about  ``Feature``, please refer to `Feature API <../reference/api.html#module-qlib.data.base>`_.

Filter
-------------------
``Qlib`` provides `NameDFilter` and `ExpressionDFilter` to filter the instruments according to users' needs.

- `NameDFilter`
    Name dynamic instrument filter. Filter the instruments based on a regulated name format. A name rule regular expression is required.

- `ExpressionDFilter`
    Expression dynamic instrument filter. Filter the instruments based on a certain expression. An expression rule indicating a certain feature field is required.
    
    - `basic features filter`: rule_expression = '$close/$open>5'
    - `cross-sectional features filter` : rule_expression = '$rank($close)<10'
    - `time-sequence features filter`: rule_expression = '$Ref($close, 3)>100'

To know more about ``Filter``, please refer to `Filter API <../reference/api.html#module-qlib.data.filter>`_.


Reference
-------------

To know more about ``Data API``, please refer to `Data API <../reference/api.html#data>`_.

Data Handler
=================

Users can use ``Data Handler`` in an automatic workflow by ``Estimator``, refer to `Estimator: Workflow Management <estimator.html>`_ for more details. 

Also, ``Data Handler`` can be used as an independent module, by which users can easily preprocess data(standardization, remove NaN, etc.) and build datasets. It is a subclass of ``qlib.contrib.estimator.handler.BaseDataHandler``, which provides some interfaces as follows.

Base Class & Interface
----------------------

Qlib provides a base class `qlib.contrib.estimator.BaseDataHandler <../reference/api.html#qlib.contrib.estimator.handler.BaseDataHandler>`_, which provides the following interfaces:

- `setup_feature`    
    Implement the interface to load the data features.

- `setup_label`   
    Implement the interface to load the data labels and calculate the users' labels. 

- `setup_processed_data`    
    Implement the interface for data preprocessing, such as preparing feature columns, discarding blank lines, and so on.

Qlib also provides two functions to help users init the data handler, users can override them for users' needs.

- `_init_kwargs`
    Users can init the kwargs of the data handler in this function, some kwargs may be used when init the raw df.
    Kwargs are the other attributes in data.args, like dropna_label, dropna_feature

- `_init_raw_df`
    Users can init the raw df, feature names, and label names of data handler in this function. 
    If the index of feature df and label df are not same, users need to override this method to merge them (e.g. inner, left, right merge).

If users want to load features and labels by config, users can inherit ``qlib.contrib.estimator.handler.ConfigDataHandler``, ``Qlib`` also have provided some preprocess method in this subclass.
If users want to use qlib data, `QLibDataHandler` is recommended. Users can inherit their custom class from `QLibDataHandler`, which is also a subclass of `ConfigDataHandler`.


Usage
--------------

``Data Handler`` can be used as a single module, which provides the following mehtods:

- `get_split_data`
    - According to the start and end dates, return features and labels of the pandas DataFrame type used for the 'Model'

- `get_rolling_data`
    - According to the start and end dates, and `rolling_period`, an iterator is returned, which can be used to traverse the features and labels used for rolling.




Example
--------------

``Data Handler`` can be run with ``estimator`` by modifying the configuration file, and can also be used as a single module. 

Know more about how to run ``Data Handler`` with ``Estimator``, please refer to `Estimator: Workflow Management <estimator.html>`_

Qlib provides implemented data handler `QLibDataHandlerClose`. The following example shows how to run `QLibDataHandlerV1` as a single module. 

.. note:: Users need to initialize ``Qlib`` with `qlib.init` first, please refer to `initialization <../start/initialization.html>`_.


.. code-block:: Python

    from qlib.contrib.estimator.handler import QLibDataHandlerClose
    from qlib.contrib.model.gbdt import LGBModel

    DATA_HANDLER_CONFIG = {
        "dropna_label": True,
        "start_date": "2007-01-01",
        "end_date": "2020-08-01",
        "market": "csi300",
    }

    TRAINER_CONFIG = {
        "train_start_date": "2007-01-01",
        "train_end_date": "2014-12-31",
        "validate_start_date": "2015-01-01",
        "validate_end_date": "2016-12-31",
        "test_start_date": "2017-01-01",
        "test_end_date": "2020-08-01",
    }

    exampleDataHandler = QLibDataHandlerClose(**DATA_HANDLER_CONFIG)

    # example of 'get_split_data'
    x_train, y_train, x_validate, y_validate, x_test, y_test = exampleDataHandler.get_split_data(**TRAINER_CONFIG)

    # example of 'get_rolling_data'

    for (x_train, y_train, x_validate, y_validate, x_test, y_test) in exampleDataHandler.get_rolling_data(**TRAINER_CONFIG):
        print(x_train, y_train, x_validate, y_validate, x_test, y_test) 


.. note:: (x_train, y_train, x_validate, y_validate, x_test, y_test) can be used as arguments for the `fit`, `predic``, and `score` methods of the ``Interday Model`` , please refer to `Model <model.html#base-class-interface>`_.

Also, the above example has been given in ``examples.estimator.train_backtest_analyze.ipynb``.

API
---------

To know more about ``Data Handler``, please refer to `Data Handler API <../reference/api.html#module-qlib.contrib.estimator.handler>`_.

Cache
==========

``Cache`` is an optional module that helps accelerate providing data by saving some frequently-used data as cache file. ``Qlib`` provides a `Memcache` class to cache the most-frequently-used data in memory, an inheritable `ExpressionCache` class and an inheritable `DatasetCache` class.

Global Memory Cache
---------------------

`Memcache` is a global memory cache mechanism that composes of three `MemCacheUnit` instances to cache **Calendar**, **Instruments**, and **Features**. The `MemCache` is defined globally in `cache.py` as `H`. Users can use `H['c'], H['i'], H['f']` to get/set `memcache`.

.. autoclass:: qlib.data.cache.MemCacheUnit
    :members:

.. autoclass:: qlib.data.cache.MemCache
    :members:


ExpressionCache
-----------------

`ExpressionCache` is a cache mechanism that saves expressions such as **Mean($close, 5)**. Users can inherit this base class to define their own cache mechanism that saves expressions according to the following steps.

- Override `self._uri` method to define how the cache file path is generated
- Override `self._expression` method to define what data will be cached and how to cache it.

The following shows the details about the interfaces:

.. autoclass:: qlib.data.cache.ExpressionCache
    :members:

``Qlib`` has currently provided implemented disk cache `DiskExpressionCache` which inherits from `ExpressionCache` . The expressions data will be stored in the disk.

DatasetCache
-----------------

`DatasetCache` is a cache mechanism that saves datasets. A certain dataset is regulated by a stock pool configuration (or a series of instruments, though not recommended), a list of expressions or static feature fields, the start time, and end time for the collected features and the frequency. Users can inherit this base class to define their own cache mechanism that saves datasets according to the following steps.

- Override `self._uri` method to define how their cache file path is generated
- Override `self._expression` method to define what data will be cached and how to cache it.

The following shows the details about the interfaces:

.. autoclass:: qlib.data.cache.DatasetCache
    :members:

``Qlib`` has currently provided implemented disk cache `DiskDatasetCache` which inherits from `DatasetCache` . The datasets data will be stored in the disk.



Data and Cache File Structure
==================================

We've specially designed a file structure to manage data and cache, please refer to the `File storage design section in Qlib paper <https://arxiv.org/abs/2009.11189>`_ for detailed information.The file structure of data and cache is listed as follows.

.. code-block:: json

    - data/
        [raw data] updated by data providers
        - calendars/
            - day.txt
        - instruments/
            - all.txt
            - csi500.txt
            - ...
        - features/
            - sh600000/
                - open.day.bin
                - close.day.bin
                - ...
            - ...
        [cached data] updated when raw data is updated
        - calculated features/
            - sh600000/
                - [hash(instrtument, field_expression, freq)]
                    - all-time expression -cache data file
                    - .meta : an assorted meta file recording the instrument name, field name, freq, and visit times
            - ...
        - cache/
            - [hash(stockpool_config, field_expression_list, freq)]
                - all-time Dataset-cache data file
                - .meta : an assorted meta file recording the stockpool config, field names and visit times
                - .index : an assorted index file recording the line index of all calendars
            - ...

