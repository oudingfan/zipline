.. _data-bundles:

数据 Bundle
------------

一个 bundle 包含价格数据、复权数据和资产数据库。
Bundle 允许我们预先加载回测所需的所有数据，并保存以备未来使用。

.. _bundles-command:

所有可用 Bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Zipline 自带了一些 bundle ，也提供添加新 bundle 的功能。
要查看可用的 bundle ，可以运行 ``bundles`` 命令，例如：

.. code-block:: bash

   $ zipline bundles
   my-custom-bundle 2016-05-05 20:35:19.809398
   my-custom-bundle 2016-05-05 20:34:53.654082
   my-custom-bundle 2016-05-05 20:34:48.401767
   quandl <no ingestions>
   quantopian-quandl 2016-05-05 20:06:40.894956

此处显示有3个可用的 bundle ：

- ``my-custom-bundle`` （用户添加）
- ``quandl`` （Zipline提供）
- ``quantopian-quandl`` （Zipline提供）

名称旁的时间显示是 bundle 的下载时间。
``my-custom-bundle`` 已经下载了三次。
``quandl`` 从未下载过，所以显示为 ``<no ingestions>`` 。
最后的 ``quantopian-quandl`` 下载过一次。

.. _ingesting-data:

下载数据
~~~~~~~~~~~~~~

使用 bundle 的第一步是进行下载。下载过程中将调用一些
bundle 命令，命令会将数据写入 Zipline 指定的位置。
默认情况下，数据将被写在 ``$ZIPLINE_ROOT/data/<bundle>``
（默认情况下 ``ZIPLINE_ROOT=~/.zipline`` ）。
数据获取可能需要一些时间，因为要下载和处理大量数据。你需要一个
`Quandl <https://docs.quandl.com/docs#section-authentication>`__
API key 来进行下载。拿到 key 之后可以这样运行：

.. code-block:: bash

   $ QUANDL_API_KEY=<yourkey> zipline ingest [-b <bundle>]


``<bundle>`` 是要下载的 bundle 的名称，默认为 ``quandl`` 。

旧数据
~~~~~~~~

使用 ``ingest`` 命令时，它会将新数据写入 ``$ZIPLINE_ROOT/data/<bundle>``
，并以当前日期命名。因为以日期命名，这让您可以按日期查看旧数据，或用旧数据进行回测。
使用旧数据进行回测，可以更容易地重现回测结果。

默认情况下会保存所有历史数据，这样的缺点是数据目录会非常大。
如前所示，我们可以使用 :ref:`bundles 命令 <bundles-command>`
列出所有已下载的数据。为了清理这些数据，有一个命令 ``clean``
，它能根据时间清理过期 bundle 。

例如：

.. code-block:: bash

   # clean everything older than <date>
   $ zipline clean [-b <bundle>] --before <date>

   # clean everything newer than <date>
   $ zipline clean [-b <bundle>] --after <date>

   # keep everything in the range of [before, after] and delete the rest
   $ zipline clean [-b <bundle>] --before <date> --after <after>

   # clean all but the last <int> runs
   $ zipline clean [-b <bundle>] --keep-last <int>


使用 bundle 进行回测
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

现在已经拿到了数据，我们可以使用 ``run`` 命令来进行回测。
可以像这样用 ``--bundle`` 参数来指定 bundle ：

.. code-block:: bash

   $ zipline run --bundle <bundle> --algofile algo.py ...


我们还可以指定 ``--bundle-timestamp`` 选项来查找 bundle 。
设置 ``--bundle-timestamp`` 会让 ``run``
使用小于等于所指定日期的最新 bundle ，这就是我们使用旧数据进行回测的方式。
``--bundle-timestamp`` 默认使用当天日期，结果就是会使用最新数据。

默认 Bundle
~~~~~~~~~~~~~~~~~~~~

.. _quandl-data-bundle:

Quandl WIKI Bundle
``````````````````

默认情况下，Zipline 自带了 ``quandl`` bundle ，它使用 quandl 的
`WIKI dataset <https://www.quandl.com/data/WIKI>`_ 。
quandl bundle 包括日线价格数据、拆股、现金股息和资产元数据。
要使用 ``quandl`` bundle 的话，我们建议在 quandl.com 上创建一个帐户
并获取 API key，这样就能使用更多 API 请求次数。有了 API key 可以这样运行：

.. code-block:: bash

   $ QUANDL_API_KEY=<api-key> zipline ingest -b quandl

在不登录（没有 API key）的情况下运行 ``ingest`` ，
我们可以设置 ``QUANDL_DOWNLOAD_ATTEMPTS`` 环境变量，
来指定从 quandl 服务器下载数据的尝试次数。默认情况下，
``QUANDL_DOWNLOAD_ATTEMPTS`` 设置为5，意思是每次请求尝试5次。

.. note::

   ``QUANDL_DOWNLOAD_ATTEMPTS`` 不是总次数，
   而是每个请求允许的失败次数。quandl 下载器为
   每 100 个股票请求一次元数据，后跟一个股票数据下载请求。


创建新的 Bundle
~~~~~~~~~~~~~~~~~~~~

Bundle 功能让 Zipline 方便使用来自各处的数据。
您也可以通过实现 ``ingest`` 函数，编写添加自己的 bundle 。

``ingest`` 函数负责将数据加载到内存，并将数据传递给 Zipline
提供的一组 writer 对象，以将数据转换为 Zipline 的内部格式。
ingest 函数可以下载像 ``quandl`` 这样的 bundle ，也可以直接加载
本机的文件。ingest 函数支持事务，以便将数据完整地写到正确位置。
如果部分失败，将不会写入不完整信息。

ingest 的声明如下：

.. code-block:: python

   ingest(environ,
          asset_db_writer,
          minute_bar_writer,
          daily_bar_writer,
          adjustment_writer,
          calendar,
          start_session,
          end_session,
          cache,
          show_progress,
          output_dir)

``environ``
```````````

``environ`` 表示要使用的环境变量的字典。通过它可以传入需要的环境变量。
如 ``quandl`` 需要传入 API key 和最大下载重试次数。

``asset_db_writer``
```````````````````

``asset_db_writer`` 是 :class:`~zipline.assets.AssetDBWriter` 的实例。
这个 writer 是为资产元数据提供的，它提供资产生命周期和符号到资产 ID（sid）的映射。
其中也可能包含资产名称、交易所和其他一些字段。要写入数据，请调用
:meth:`~zipline.assets.AssetDBWriter.write` 并传入包含各种元数据的 DataFrame。
有关数据格式的更多信息，可在 writer 的文档中找到。

``minute_bar_writer``
`````````````````````

``minute_bar_writer`` 是 :class:`~zipline.data.minute_bars.BcolzMinuteBarWriter` 的实例。
这个 writer 用来将数据转化为 Zipline 内部可识别的 bcolz 格式，以供
:class:`~zipline.data.minute_bars.BcolzMinuteBarReader` 读取。
如果提供了分钟级数据，用户需调用 :meth:`~zipline.data.minute_bars.BcolzMinuteBarWriter.write`
并传入 [(sid, DataFrame)] 类型的 tuple ，第二个参数则是 ``show_progress`` 。
如果数据源没提供分钟级数据，则没必要调用这个 write 函数，或向
:meth:`~zipline.data.minute_bars.BcolzMinuteBarWriter.write`
第一个参数传递空 list ，来表明没有分钟级数据。

.. note::

   可以将生成器和迭代器传递给 :meth:`~zipline.data.minute_bars.BcolzMinuteBarWriter.write`
   以免分钟级数据占用过多内存。日期严格增加的话，sid 也可能在数据中出现多次。

``daily_bar_writer``
````````````````````

``daily_bar_writer`` 是 :class:`~zipline.data.bcolz_daily_bars.BcolzDailyBarWriter` 的实例。
这个 writer 用来将数据转化为 Zipline 内部可识别的 bcolz 格式，以供
:class:`~zipline.data.bcolz_daily_bars.BcolzDailyBarReader` 读取。
如果提供了日线级数据，用户需调用 :meth:`~zipline.data.minute_bars.BcolzDailyBarWriter.write`
并传入 [(sid, DataFrame)] 类型的 tuple ，第二个参数则是 ``show_progress`` 。
如果数据源没提供日线级数据，则没必要调用这个 write 函数，或向
:meth:`~zipline.data.minute_bars.BcolzMinuteBarWriter.write`
第一个参数传递空 list ，来表明没有日线级数据。
如果提供了分钟级数据而没有提供日线数据，则会由分钟级数据生成日线数据。

.. note::

   和 ``minute_bar_writer`` 相同，可以将生成器和迭代器传递给
   :meth:`~zipline.data.minute_bars.BcolzMinuteBarWriter.write` 以免占用过多内存。
   和 ``minute_bar_writer`` 不同的是，一个 sid 只会出现一次。

``adjustment_writer``
`````````````````````

``adjustment_writer`` 是
:class:`~zipline.data.adjustments.SQLiteAdjustmentWriter` 的实例。
这个 writer 使用来保存分股、合股、股息分红的。
数据应该以 DataFrame 的格式传递给
:meth:`~zipline.data.adjustments.SQLiteAdjustmentWriter.write` 。
所有参数都是可选的，如果您的数据量很大，这个 writer 也都可以接收。

``calendar``
````````````

``calendar`` 是 :class:`zipline.utils.calendars.TradingCalendar` 的实例。
calendar 用来帮助生成和日期有关的查询。

``start_session``
`````````````````

``start_session`` 是 :class:`pandas.Timestamp` 的实例，
表示 bundle 加载的起始日期。

``end_session``
```````````````

``end_session`` 是 :class:`pandas.Timestamp` 的实例，
表示 bundle 加载的结束日期。

``cache``
`````````

``cache`` 是 :class:`~zipline.utils.cache.dataframe_cache` 的实例。
它是一个 string 到 DataFrame 映射的字典结构。
它的作用是对 ingest 过程出现的错误退出提供支持。
基本逻辑是， ingest 首先检查是否有缓存，没有的话，获取原始数据并保存到缓存中，然后解析并写入数据。
只有在成功加载后才会清除缓存，这可以防止在解析中出现错误而需要重新下载所有数据。
如果获取数据非常快，例如加载本地文件，那就不需要使用缓存功能。

``show_progress``
`````````````````

``show_progress`` 是一个布尔型变量，它表示用户是否希望看到下载和写入数据的进度。
在一些例子中，进度条指示已下载文件数比例，还有一些指示文件转换进度。
能够帮助实现 ``show_progress`` 循环功能的工具是
:class:`~zipline.utils.cli.maybe_show_progress` 。
这个参数应该始终被转发给 ``minute_bar_writer.write`` 和 ``daily_bar_writer.write`` 。


``output_dir``
``````````````

``output_dir`` 是一个字符型变量，表示数据写入的位置。
``output_dir`` 是 ``$ZIPLINE_ROOT`` 的子目录，它包含了运行的开始时间。
如果由于某些原因，不使用 writers 保存数据，可以这部分数据存放到这里。
例如 ``quantopian:quandl`` 将数据解压到了 ``output_dir`` 。

从.csv文件中加载数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Zipline 提供了一个名为 ``csvdir`` 的包，它允许用户从
``.csv`` 文件加载数据。文件的格式应为 OHLCV 格式，
并带有带日期、分红和分股。下面提供了一个例子，更多例子可以在
``zipline/tests/resources/csvdir_samples`` 找到。

.. code-block:: text

	 date,open,high,low,close,volume,dividend,split
	 2012-01-03,58.485714,58.92857,58.42857,58.747143,75555200,0.0,1.0
	 2012-01-04,58.57143,59.240002,58.468571,59.062859,65005500,0.0,1.0
	 2012-01-05,59.278572,59.792858,58.952858,59.718571,67817400,0.0,1.0
	 2012-01-06,59.967144,60.392857,59.888573,60.342857,79573200,0.0,1.0
	 2012-01-09,60.785713,61.107143,60.192856,60.247143,98506100,0.0,1.0
	 2012-01-10,60.844284,60.857143,60.214287,60.462856,64549100,0.0,1.0
	 2012-01-11,60.382858,60.407143,59.901428,60.364285,53771200,0.0,1.0

组织好上面格式的数据之后，您可以编辑 ``~/.zipline/extension.py``
导入 csvdir 和 ``pandas`` 包。

.. code-block:: python

	 import pandas as pd

	 from zipline.data.bundles import register
	 from zipline.data.bundles.csvdir import csvdir_equities

然后，指定开始和结束时间：

.. code-block:: python

	 start_session = pd.Timestamp('2016-1-1', tz='utc')
	 end_session = pd.Timestamp('2018-1-1', tz='utc')

然后我们可以传入 ``.csv`` 文件路径，用 ``register()``
注册我们的自己编写的 bundle ：

.. code-block:: python

    register(
        'custom-csvdir-bundle',
        csvdir_equities(
            ['daily'],
            '/path/to/your/csvs',
        ),
        calendar_name='NYSE', # US equities
        start_session=start_session,
        end_session=end_session
    )

运行命令导入自己编写的 bundle ：

.. code-block:: bash

	 $ zipline ingest -b custom-csvdir-bundle
	 Loading custom pricing data:   [############------------------------]   33% | FAKE: sid 0
	 Loading custom pricing data:   [########################------------]   66% | FAKE1: sid 1
	 Loading custom pricing data:   [####################################]  100% | FAKE2: sid 2
	 Loading custom pricing data:   [####################################]  100%
	 Merging daily equity files:  [####################################]

	 # optionally, we can pass the location of our csvs via the command line
	 $ CSVDIR=/path/to/your/csvs zipline ingest -b custom-csvdir-bundle


如果您想使用不在纽约证券交易所和 Zipline 日历中的股票数据，您可以参照
``交易日历`` 章节来建立您的自定义交易日历，并使用 ``register()`` 导入。