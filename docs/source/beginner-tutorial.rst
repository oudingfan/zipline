入门指南
-------------------------

基本概念
~~~~~~~~~~

Zipline 是一个用 Python 开发的开源的算法交易框架。

源代码可以在此查看： https://github.com/quantopian/zipline

主要优点有：

-  模拟现实：滑点、手续费、延时成交。
-  流式处理：单独处理每个事件，避免未来函数。
-  丰富的库：技术指标（如移动平均）、风险评估（如夏普比率）。
-  由 `Quantopian <https://www.quantopian.com>`__ 开发和持续维护，
   为 Zipline 提供易于使用的网络界面。Quantopian 还提供 10
   年的分钟级美国股市数据和实时交易功能。本教程针对不使用 Quantopian
   的 Zipline 用户。如果想了解 Quantopian，请参阅
   `此处 <https://www.quantopian.com/faq#get-started>`__ 。

本教程假设您已正确安装了 Zipline，如果尚未安装 Zipline ，请参阅
`安装指南 <./install.html>`__ 。

每个 ``zipline`` 策略都必须包含以下两个函数：

* ``initialize(context)``
* ``handle_data(context, data)``

在执行策略之前， ``zipline`` 调用 ``initialize()`` 函数并传入一个
``context`` 变量。``context`` 是一个持久变量，用于存储策略迭代中需要访问的数据。

策略被初始化之后，``zipline`` 为每个事件调用一次 ``handle_data()`` 函数。
每次调用时，都会传入相同的 ``context`` 变量和 ``data`` 事件框架，``data``
中含有当前开盘价、最高价、最低价、收盘价（OHLC）和成交量的数据。有关这些函数的更多信息，请参阅
`Quantopian 相关文档 <https://www.quantopian.com/help#api-toplevel>`__.

第一个策略
~~~~~~~~~~~~~~~~~~

让我们看下 ``examples`` 文件夹中的一个简单例子，``buyapple.py``：

.. code-block:: python

   from zipline.api import order, record, symbol


   def initialize(context):
       pass


   def handle_data(context, data):
       order(symbol('AAPL'), 10)
       record(AAPL=data.current(symbol('AAPL'), 'price'))


可以看到，首先我们引入一些需要的函数。策略中所有常用的函数可以在 ``zipline.api`` 中找到。
这里的 :func:`~zipline.api.order()` 函数接受两个参数：一个股票代码，一个数量（为负则为卖出或做空）。
在例子中，每次迭代我们买入 10 股 APPL 股票。更多 ``order()`` 信息，可参阅
`Quantopian 文档 <https://www.quantopian.com/help#api-order>`__ 。

:func:`~zipline.api.record` 函数能在每一次迭代的时候记录一些临时数据。
第一个参数接受 ``变量名=变量值`` 的这样参数。回测结束进行分析的时候，能够通过变量名访问到
这些记录的数据（下面会有示例）。您还可以看到如何在 ``data`` 事件框架中访问之前记录的
AAPL 股票的价格数据（更多信息请访问
`这里 <https://www.quantopian.com/help#api-event-properties>`__ ）。

运行策略
~~~~~~~~~~~~~~~~~~~~~

现在要在价格数据上面运行这个策略， ``zipline`` 提供三种接口：命令行、``IPython Notebook``
和 :func:`~zipline.run_algorithm` 。

获取数据
^^^^^^^^^^^^^^
如果您还没有拿到数据，您将需要 `Quandl <https://docs.quandl.com/docs#section-authentication>`__
的 API key 来获取默认数据。然后运行：

.. code-block:: bash

   $ QUANDL_API_KEY=<yourkey> zipline ingest [-b <bundle>]

``<bundle>`` 是要获取数据的名称，默认是 ``quandl`` 。

您可以查看 :ref:`获取数据 <ingesting-data>` 获取更多信息。

命令行
^^^^^^^^^^^^^^^^^^^^^^

安装 Zipline 之后，您就可以在命令行中运行以下命令了（在
Windows 中使用 ``cmd.exe`` ，在 macOS 中使用 Terminal App）：

.. code-block:: bash

   $ zipline run --help

.. parsed-literal::

  Usage: zipline run [OPTIONS]

  Run a backtest for the given algorithm.

  Options:
   -f, --algofile FILENAME         The file that contains the algorithm to run.
   -t, --algotext TEXT             The algorithm script to run.
   -D, --define TEXT               Define a name to be bound in the namespace
                                   before executing the algotext. For example
                                   '-Dname=value'. The value may be any python
                                   expression. These are evaluated in order so
                                   they may refer to previously defined names.
   --data-frequency [daily|minute]
                                   The data frequency of the simulation.
                                   [default: daily]
   --capital-base FLOAT            The starting capital for the simulation.
                                   [default: 10000000.0]
   -b, --bundle BUNDLE-NAME        The data bundle to use for the simulation.
                                   [default: quandl]
   --bundle-timestamp TIMESTAMP    The date to lookup data on or before.
                                   [default: <current-time>]
   -s, --start DATE                The start date of the simulation.
   -e, --end DATE                  The end date of the simulation.
   -o, --output FILENAME           The location to write the perf data. If this
                                   is '-' the perf will be written to stdout.
                                   [default: -]
   --trading-calendar TRADING-CALENDAR
                                   The calendar you want to use e.g. LSE. NYSE
                                   is the default.
   --print-algo / --no-print-algo  Print the algorithm to stdout.
   --help                          Show this message and exit.

您可以看到，由很多可以设置的参数，如 ``-f`` 设置使用的策略， ``-b`` 设置使用的数据（默认为 ``quandl``）。
也可以用 ``--start`` 和 ``--end`` 指定回测时间区间。
您还可以保存回测结果以便后续分析，方法是用 ``--output`` 参数指定文件名，结果将以 ``DateFrame``
的格式保存到 Python 的 pickle 文件内。您还可以将配置写到配置文件里，然后用 ``-c``
指定配置文件的路径，这样就不用每次都指定各种参数了（请参阅示例目录中的 .conf 文件）。

要执行上面的算法，并把结果保存到 ``buyapple_out.pickle`` ，可以像这样调用 ``zipline run`` ：

.. code-block:: python

    zipline run -f ../../zipline/examples/buyapple.py --start 2016-1-1 --end 2018-1-1 -o buyapple_out.pickle


.. parsed-literal::

    AAPL
    [2018-01-03 04:30:50.150039] WARNING: Loader: Refusing to download new benchmark data because a download succeeded at 2018-01-03 04:01:34+00:00.
    [2018-01-03 04:30:50.191479] WARNING: Loader: Refusing to download new treasury data because a download succeeded at 2018-01-03 04:01:35+00:00.
    [2018-01-03 04:30:51.843465] INFO: Performance: Simulated 503 trading days out of 503.
    [2018-01-03 04:30:51.843598] INFO: Performance: first open: 2016-01-04 14:31:00+00:00
    [2018-01-03 04:30:51.843672] INFO: Performance: last close: 2017-12-29 21:00:00+00:00


``run`` 首先调用 ``initialize()`` 函数，然后逐条将股票价格传入 ``handle_data()`` 。
每次调用 ``handle_data()`` 后，都会买入 10 股 AAPL 。
调用 ``order()`` 后， ``Zipline`` 记录下单信息并送入盘口。
在 ``handle_data()`` 函数结束后， ``Zipline`` 查找未成交订单，并进行撮合交易。
如果市场容量足够容纳您的订单，则会在添加佣金并应用滑点模型后成交订单。
滑点模型会模拟您的订单对市场价格的影响，因此收取的费用不仅仅是股票价格 \* 10 。
（您可以更改 ``Zipline`` 使用的佣金和滑点模型，请参阅
`Quantopian 文档 <https://www.quantopian.com/help#ide-slippage>`__
获取更多信息）。

让我们看一下保存的 ``DataFrame`` 结果。我们在 IPython Notebook 中用 ``pandas`` 加载结果并打印前 10 行。
``Zipline`` 大量使用 ``pandas`` ，特别是对于数据输入和输出这部分，所以值得花一些时间来学习。

.. code-block:: python

    import pandas as pd
    perf = pd.read_pickle('buyapple_out.pickle') # read in perf DataFrame
    perf.head()

.. raw:: html

    <div style="max-height: 1000px; max-width: 1500px; overflow: auto;">
    <table border="1" class="dataframe">
    <thead>
      <tr style="text-align: right;">
        <th></th>
        <th>AAPL</th>
        <th>algo_volatility</th>
        <th>algorithm_period_return</th>
        <th>alpha</th>
        <th>benchmark_period_return</th>
        <th>benchmark_volatility</th>
        <th>beta</th>
        <th>capital_used</th>
        <th>ending_cash</th>
        <th>ending_exposure</th>
        <th>ending_value</th>
        <th>excess_return</th>
        <th>gross_leverage</th>
        <th>long_exposure</th>
        <th>long_value</th>
        <th>longs_count</th>
        <th>max_drawdown</th>
        <th>max_leverage</th>
        <th>net_leverage</th>
        <th>orders</th>
        <th>period_close</th>
        <th>period_label</th>
        <th>period_open</th>
        <th>pnl</th>
        <th>portfolio_value</th>
        <th>positions</th>
        <th>returns</th>
        <th>sharpe</th>
        <th>short_exposure</th>
        <th>short_value</th>
        <th>shorts_count</th>
        <th>sortino</th>
        <th>starting_cash</th>
        <th>starting_exposure</th>
        <th>starting_value</th>
        <th>trading_days</th>
        <th>transactions</th>
        <th>treasury_period_return</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>2016-01-04 21:00:00+00:00</th>
        <td>105.35</td>
        <td>NaN</td>
        <td>0.000000e+00</td>
        <td>NaN</td>
        <td>-0.013983</td>
        <td>NaN</td>
        <td>NaN</td>
        <td>0.0</td>
        <td>10000000.0</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0.000000</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0</td>
        <td>0.000000e+00</td>
        <td>0.0</td>
        <td>0.000000</td>
        <td>[{\'dt\': 2016-01-04 21:00:00+00:00, \'reason\': N...</td>
        <td>2016-01-04 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-04 14:31:00+00:00</td>
        <td>0.0</td>
        <td>10000000.0</td>
        <td>[]</td>
        <td>0.000000e+00</td>
        <td>NaN</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>NaN</td>
        <td>10000000.0</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>1</td>
        <td>[]</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-05 21:00:00+00:00</th>
        <td>102.71</td>
        <td>0.000001</td>
        <td>-1.000000e-07</td>
        <td>-0.000022</td>
        <td>-0.012312</td>
        <td>0.175994</td>
        <td>-0.000006</td>
        <td>-1028.1</td>
        <td>9998971.9</td>
        <td>1027.1</td>
        <td>1027.1</td>
        <td>0.0</td>
        <td>0.000103</td>
        <td>1027.1</td>
        <td>1027.1</td>
        <td>1</td>
        <td>-1.000000e-07</td>
        <td>0.0</td>
        <td>0.000103</td>
        <td>[{\'dt\': 2016-01-05 21:00:00+00:00, \'reason\': N...</td>
        <td>2016-01-05 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-05 14:31:00+00:00</td>
        <td>-1.0</td>
        <td>9999999.0</td>
        <td>[{\'sid\': Equity(8 [AAPL]), \'last_sale_price\': ...</td>
        <td>-1.000000e-07</td>
        <td>-11.224972</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-11.224972</td>
        <td>10000000.0</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>2</td>
        <td>[{\'order_id\': \'4011063b5c094e82a5391527044098b...</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-06 21:00:00+00:00</th>
        <td>100.70</td>
        <td>0.000019</td>
        <td>-2.210000e-06</td>
        <td>-0.000073</td>
        <td>-0.024771</td>
        <td>0.137853</td>
        <td>0.000054</td>
        <td>-1008.0</td>
        <td>9997963.9</td>
        <td>2014.0</td>
        <td>2014.0</td>
        <td>0.0</td>
        <td>0.000201</td>
        <td>2014.0</td>
        <td>2014.0</td>
        <td>1</td>
        <td>-2.210000e-06</td>
        <td>0.0</td>
        <td>0.000201</td>
        <td>[{\'dt\': 2016-01-06 21:00:00+00:00, \'reason\': N...</td>
        <td>2016-01-06 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-06 14:31:00+00:00</td>
        <td>-21.1</td>
        <td>9999977.9</td>
        <td>[{\'sid\': Equity(8 [AAPL]), \'last_sale_price\': ...</td>
        <td>-2.110000e-06</td>
        <td>-9.823839</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-9.588756</td>
        <td>9998971.9</td>
        <td>1027.1</td>
        <td>1027.1</td>
        <td>3</td>
        <td>[{\'order_id\': \'3bf9fe20cc46468d99f741474226c03...</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-07 21:00:00+00:00</th>
        <td>96.45</td>
        <td>0.000064</td>
        <td>-1.081000e-05</td>
        <td>0.000243</td>
        <td>-0.048168</td>
        <td>0.167868</td>
        <td>0.000300</td>
        <td>-965.5</td>
        <td>9996998.4</td>
        <td>2893.5</td>
        <td>2893.5</td>
        <td>0.0</td>
        <td>0.000289</td>
        <td>2893.5</td>
        <td>2893.5</td>
        <td>1</td>
        <td>-1.081000e-05</td>
        <td>0.0</td>
        <td>0.000289</td>
        <td>[{\'dt\': 2016-01-07 21:00:00+00:00, \'reason\': N...</td>
        <td>2016-01-07 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-07 14:31:00+00:00</td>
        <td>-86.0</td>
        <td>9999891.9</td>
        <td>[{\'sid\': Equity(8 [AAPL]), \'last_sale_price\': ...</td>
        <td>-8.600019e-06</td>
        <td>-10.592737</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-9.688947</td>
        <td>9997963.9</td>
        <td>2014.0</td>
        <td>2014.0</td>
        <td>4</td>
        <td>[{\'order_id\': \'6af6aed9fbb44a6bba17e802051b94d...</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-08 21:00:00+00:00</th>
        <td>96.96</td>
        <td>0.000063</td>
        <td>-9.380000e-06</td>
        <td>0.000466</td>
        <td>-0.058601</td>
        <td>0.145654</td>
        <td>0.000311</td>
        <td>-970.6</td>
        <td>9996027.8</td>
        <td>3878.4</td>
        <td>3878.4</td>
        <td>0.0</td>
        <td>0.000388</td>
        <td>3878.4</td>
        <td>3878.4</td>
        <td>1</td>
        <td>-1.081000e-05</td>
        <td>0.0</td>
        <td>0.000388</td>
        <td>[{\'dt\': 2016-01-08 21:00:00+00:00, \'reason\': N...</td>
        <td>2016-01-08 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-08 14:31:00+00:00</td>
        <td>14.3</td>
        <td>9999906.2</td>
        <td>[{\'sid\': Equity(8 [AAPL]), \'last_sale_price\': ...</td>
        <td>1.430015e-06</td>
        <td>-7.511729</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-7.519659</td>
        <td>9996998.4</td>
        <td>2893.5</td>
        <td>2893.5</td>
        <td>5</td>
        <td>[{\'order_id\': \'18f64975732449a18fca06e9c69bf5c...</td>
        <td>0.0</td>
      </tr>
    </tbody>
    </table>
    </div>

可以看到，从 2016 年第一个交易日起，每天都有一条记录。在每列中可以找到策略表现的各类信息。
第一列的 ``AAPL`` 的值由 ``record()`` 写入，方便后续绘制图线，比较策略收益与股票价格的表现。

.. code-block:: python

    %pylab inline
    figsize(12, 12)
    import matplotlib.pyplot as plt

    ax1 = plt.subplot(211)
    perf.portfolio_value.plot(ax=ax1)
    ax1.set_ylabel('Portfolio Value')
    ax2 = plt.subplot(212, sharex=ax1)
    perf.AAPL.plot(ax=ax2)
    ax2.set_ylabel('AAPL Stock Price')

.. parsed-literal::

    Populating the interactive namespace from numpy and matplotlib

.. parsed-literal::

    <matplotlib.text.Text at 0x10c48c198>

.. image:: tutorial_files/tutorial_11_2.png


``portfolio_value`` 表示我们策略的收益，可以看到和 AAPL 的价格走势很接近。
这很好理解，因为只是简单买入。

IPython Notebook
~~~~~~~~~~~~~~~~

`IPython Notebook <http://ipython.org/notebook.html>`__ 是一个非常强大的、
基于浏览器界面的 Python 解释器（本教程就是基于它而写）。它已经是大多数宽客的事实上的开发环境，
``Zipline`` 提供了一种不使用命令行就能在 Notebook 中运行策略的方法。

使用时，将策略写在 Notebook 的一个 cell 中，并且让 ``Zipline`` 知道将要运行这个策略。
方法是在 ``import zipline`` 使用后 ``%%zipline`` 这个 IPython 魔术方法。
这个魔术方法采用与上面命令行相同的参数。为了和上面命令行的例子保持一致，在 import ``zipline``
之后，运行下面的 cell 。

.. code-block:: python

   %load_ext zipline

.. code-block:: python

   %%zipline --start 2016-1-1 --end 2018-1-1
   from zipline.api import symbol, order, record

   def initialize(context):
       pass

   def handle_data(context, data):
       order(symbol('AAPL'), 10)
       record(AAPL=data[symbol('AAPL')].price)

注意我们并没有指定输入文件，因为魔术方法将使用 cell 内的内容，并在那里找到你的策略算法。
此外，我们也没有定义输出文件，而是指定了 ``-o`` 参数，他将在环境创建一个包含像上面那样
结果的 ``DataFrame`` 变量。

.. code-block:: python

   _.head()

.. raw:: html

   <div style="max-height: 1000px; max-width: 1500px; overflow: auto;">
   <table border="1" class="dataframe">
    <thead>
      <tr style="text-align: right;">
        <th></th>
        <th>AAPL</th>
        <th>algo_volatility</th>
        <th>algorithm_period_return</th>
        <th>alpha</th>
        <th>benchmark_period_return</th>
        <th>benchmark_volatility</th>
        <th>beta</th>
        <th>capital_used</th>
        <th>ending_cash</th>
        <th>ending_exposure</th>
        <th>ending_value</th>
        <th>excess_return</th>
        <th>gross_leverage</th>
        <th>long_exposure</th>
        <th>long_value</th>
        <th>longs_count</th>
        <th>max_drawdown</th>
        <th>max_leverage</th>
        <th>net_leverage</th>
        <th>orders</th>
        <th>period_close</th>
        <th>period_label</th>
        <th>period_open</th>
        <th>pnl</th>
        <th>portfolio_value</th>
        <th>positions</th>
        <th>returns</th>
        <th>sharpe</th>
        <th>short_exposure</th>
        <th>short_value</th>
        <th>shorts_count</th>
        <th>sortino</th>
        <th>starting_cash</th>
        <th>starting_exposure</th>
        <th>starting_value</th>
        <th>trading_days</th>
        <th>transactions</th>
        <th>treasury_period_return</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>2016-01-04 21:00:00+00:00</th>
        <td>105.35</td>
        <td>NaN</td>
        <td>0.000000e+00</td>
        <td>NaN</td>
        <td>-0.013983</td>
        <td>NaN</td>
        <td>NaN</td>
        <td>0.00</td>
        <td>10000000.00</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0.000000</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>0</td>
        <td>0.000000e+00</td>
        <td>0.0</td>
        <td>0.000000</td>
        <td>[{\'created\': 2016-01-04 21:00:00+00:00, \'reaso...</td>
        <td>2016-01-04 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-04 14:31:00+00:00</td>
        <td>0.00</td>
        <td>10000000.00</td>
        <td>[]</td>
        <td>0.000000e+00</td>
        <td>NaN</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>NaN</td>
        <td>10000000.00</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>1</td>
        <td>[]</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-05 21:00:00+00:00</th>
        <td>102.71</td>
        <td>1.122497e-08</td>
        <td>-1.000000e-09</td>
        <td>-2.247510e-07</td>
        <td>-0.012312</td>
        <td>0.175994</td>
        <td>-6.378047e-08</td>
        <td>-1027.11</td>
        <td>9998972.89</td>
        <td>1027.1</td>
        <td>1027.1</td>
        <td>0.0</td>
        <td>0.000103</td>
        <td>1027.1</td>
        <td>1027.1</td>
        <td>1</td>
        <td>-9.999999e-10</td>
        <td>0.0</td>
        <td>0.000103</td>
        <td>[{\'created\': 2016-01-04 21:00:00+00:00, \'reaso...</td>
        <td>2016-01-05 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-05 14:31:00+00:00</td>
        <td>-0.01</td>
        <td>9999999.99</td>
        <td>[{\'amount\': 10, \'cost_basis\': 102.711000000000...</td>
        <td>-1.000000e-09</td>
        <td>-11.224972</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-11.224972</td>
        <td>10000000.00</td>
        <td>0.0</td>
        <td>0.0</td>
        <td>2</td>
        <td>[{\'dt\': 2016-01-05 21:00:00+00:00, \'order_id\':...</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-06 21:00:00+00:00</th>
        <td>100.70</td>
        <td>1.842654e-05</td>
        <td>-2.012000e-06</td>
        <td>-4.883861e-05</td>
        <td>-0.024771</td>
        <td>0.137853</td>
        <td>5.744807e-05</td>
        <td>-1007.01</td>
        <td>9997965.88</td>
        <td>2014.0</td>
        <td>2014.0</td>
        <td>0.0</td>
        <td>0.000201</td>
        <td>2014.0</td>
        <td>2014.0</td>
        <td>1</td>
        <td>-2.012000e-06</td>
        <td>0.0</td>
        <td>0.000201</td>
        <td>[{\'created\': 2016-01-05 21:00:00+00:00, \'reaso...</td>
        <td>2016-01-06 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-06 14:31:00+00:00</td>
        <td>-20.11</td>
        <td>9999979.88</td>
        <td>[{\'amount\': 20, \'cost_basis\': 101.706000000000...</td>
        <td>-2.011000e-06</td>
        <td>-9.171989</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-9.169708</td>
        <td>9998972.89</td>
        <td>1027.1</td>
        <td>1027.1</td>
        <td>3</td>
        <td>[{\'dt\': 2016-01-06 21:00:00+00:00, \'order_id\':...</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-07 21:00:00+00:00</th>
        <td>96.45</td>
        <td>6.394658e-05</td>
        <td>-1.051300e-05</td>
        <td>2.633450e-04</td>
        <td>-0.048168</td>
        <td>0.167868</td>
        <td>3.005102e-04</td>
        <td>-964.51</td>
        <td>9997001.37</td>
        <td>2893.5</td>
        <td>2893.5</td>
        <td>0.0</td>
        <td>0.000289</td>
        <td>2893.5</td>
        <td>2893.5</td>
        <td>1</td>
        <td>-1.051300e-05</td>
        <td>0.0</td>
        <td>0.000289</td>
        <td>[{\'created\': 2016-01-06 21:00:00+00:00, \'reaso...</td>
        <td>2016-01-07 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-07 14:31:00+00:00</td>
        <td>-85.01</td>
        <td>9999894.87</td>
        <td>[{\'amount\': 30, \'cost_basis\': 99.9543333333335...</td>
        <td>-8.501017e-06</td>
        <td>-10.357397</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-9.552189</td>
        <td>9997965.88</td>
        <td>2014.0</td>
        <td>2014.0</td>
        <td>4</td>
        <td>[{\'dt\': 2016-01-07 21:00:00+00:00, \'order_id\':...</td>
        <td>0.0</td>
      </tr>
      <tr>
        <th>2016-01-08 21:00:00+00:00</th>
        <td>96.96</td>
        <td>6.275294e-05</td>
        <td>-8.984000e-06</td>
        <td>4.879306e-04</td>
        <td>-0.058601</td>
        <td>0.145654</td>
        <td>3.118401e-04</td>
        <td>-969.61</td>
        <td>9996031.76</td>
        <td>3878.4</td>
        <td>3878.4</td>
        <td>0.0</td>
        <td>0.000388</td>
        <td>3878.4</td>
        <td>3878.4</td>
        <td>1</td>
        <td>-1.051300e-05</td>
        <td>0.0</td>
        <td>0.000388</td>
        <td>[{\'created\': 2016-01-07 21:00:00+00:00, \'reaso...</td>
        <td>2016-01-08 21:00:00+00:00</td>
        <td>2016-01</td>
        <td>2016-01-08 14:31:00+00:00</td>
        <td>15.29</td>
        <td>9999910.16</td>
        <td>[{\'amount\': 40, \'cost_basis\': 99.2060000000002...</td>
        <td>1.529016e-06</td>
        <td>-7.215497</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>-7.301134</td>
        <td>9997001.37</td>
        <td>2893.5</td>
        <td>2893.5</td>
        <td>5</td>
        <td>[{\'dt\': 2016-01-08 21:00:00+00:00, \'order_id\':...</td>
        <td>0.0</td>
      </tr>
    </tbody>
   </table>
   </div>

使用 ``history`` 回看价格数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

示例：双移动平均线交叉
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

双移动平均线交叉（DMA）一个经典的动量策略。要求较高的交易者可能已经不再使用它了，
但它仍旧有一些启发性。基本思想是引入两条移动平均线（mavg），一条长周期的捕捉长期趋势，
一条短周期的捕获短期趋势。短线上穿长线，我们认为将持续上涨，因此做多。下穿的时候我们
认为还将下跌，因此卖出。

因为计算移动平均线需要用到历史数据，因此引入一个新概念：History 。

``data.history()`` 函数可以让你很方便的获取历史数据。第一个参数是你需要获取的数据条数，
第二个是时间周期（如 ``1d`` 或 ``1m``，使用 ``1m`` 时要提供分钟级数据）。有关
``history()`` 更多说明，请参阅
`Quantopian 文档 <https://www.quantopian.com/help#ide-history>`__ 。
让我们看一个策略的例子：

.. code-block:: python

   %%zipline --start 2014-1-1 --end 2018-1-1 -o dma.pickle


   from zipline.api import order_target, record, symbol
   import matplotlib.pyplot as plt

   def initialize(context):
       context.i = 0
       context.asset = symbol('AAPL')


   def handle_data(context, data):
       # Skip first 300 days to get full windows
       context.i += 1
       if context.i < 300:
           return

       # Compute averages
       # data.history() has to be called with the same params
       # from above and returns a pandas dataframe.
       short_mavg = data.history(context.asset, 'price', bar_count=100, frequency="1d").mean()
       long_mavg = data.history(context.asset, 'price', bar_count=300, frequency="1d").mean()

       # Trading logic
       if short_mavg > long_mavg:
           # order_target orders as many shares as needed to
           # achieve the desired number of shares.
           order_target(context.asset, 100)
       elif short_mavg < long_mavg:
           order_target(context.asset, 0)

       # Save values for later inspection
       record(AAPL=data.current(context.asset, 'price'),
              short_mavg=short_mavg,
              long_mavg=long_mavg)


   def analyze(context, perf):
       fig = plt.figure()
       ax1 = fig.add_subplot(211)
       perf.portfolio_value.plot(ax=ax1)
       ax1.set_ylabel('portfolio value in $')

       ax2 = fig.add_subplot(212)
       perf['AAPL'].plot(ax=ax2)
       perf[['short_mavg', 'long_mavg']].plot(ax=ax2)

       perf_trans = perf.ix[[t != [] for t in perf.transactions]]
       buys = perf_trans.ix[[t[0]['amount'] > 0 for t in perf_trans.transactions]]
       sells = perf_trans.ix[
           [t[0]['amount'] < 0 for t in perf_trans.transactions]]
       ax2.plot(buys.index, perf.short_mavg.ix[buys.index],
                '^', markersize=10, color='m')
       ax2.plot(sells.index, perf.short_mavg.ix[sells.index],
                'v', markersize=10, color='k')
       ax2.set_ylabel('price in $')
       plt.legend(loc=0)
       plt.show()

.. image:: tutorial_files/tutorial_22_1.png

这里我们定义了一个 ``analyze()`` 函数，在回测结束后它将自动被调用
（在 Quantopian 这个功能还不可用）。

虽然收益变化不明显，但 ``history()`` 的作用不容小觑，因为大部分策略都是基于历史数据的。
很容易使用 `scikit-learn <http://scikit-learn.org/stable/>`__ 设计一个分类器，
基于历史数据来预测未来市场的走向（注意，大部分 ``scikit-learn`` 函数不使用
``pandas.DataFrame`` ，而是使用 ``numpy.ndarray``，所以你可以直接使用
``DataFrame`` 的原始值 ``.values`` ，它是 ``ndarray`` 格式的）。

上面我还使用了 ``order_target()`` 函数。这类函数能使订单管理和收益再平衡变得容易。
更多信息，请参阅
`order 函数文档 <https://www.quantopian.com/help#api-order-methods>`__ 。

结论
~~~~~~~~~~~

希望这个教程能让你大致了解 ``Zipline`` 的结构、API 和功能。下一步，可以查看一些
`示例 <https://github.com/quantopian/zipline/tree/master/zipline/examples>`__ 。

可以在 `邮件列表 <https://groups.google.com/forum/#!forum/zipline>`__ 中提问，在
`GitHub issue <https://github.com/quantopian/zipline/issues?state=open>`__ 和
`Quantopian <https://quantopian.com>`__  中提交问题。
