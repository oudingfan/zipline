.. _metrics:

风险和绩效指标
----------------------------

风险和绩效指标总结了 Zipline 的回测结果。
这些指标可以度量策略的表现，如收益、现金流、风险度、波动性、
beta 值。度量指标能以分钟级、日线级或在回测完成时报告。
一个指标可以在多个时间维度上进行报告。

指标集
~~~~~~~~~~~~

Zipline 将风险和绩效指标打包为“指标集”。一个指标集定义了在一次回测中所有的指标。
指标集包含不同时间维度的指标。默认指标集将计算一系列指标，如策略收益、波动率、夏普比率和 beta 值。

选择指标集
~~~~~~~~~~~~~~~~~~~~~~~~~

运行一次回测时，用户可以选择指标集来查看回测报告。怎样选择指标集取决于运行策略的环境。

命令行和 IPython
``````````````````````````````

当运行在命令行和 IPython 下时，指标集可以通过传递 ``--metrics-set`` 参数来选择。例如：

.. code-block:: bash

   $ zipline run algorithm.py -s 2014-01-01 -e 2014-02-01 --metrics-set my-metrics-set

``run_algorithm``
`````````````````

当通过 :func:`~zipline.run_algorithm` 函数来运行回测时，指标集可以通过传递
``metrics_set`` 来选择。参数可以是指标集的名称或是实例。例如：

.. code-block:: python

   run_algorithm(..., metrics_set='my-metrics-set')
   run_algorithm(..., metrics_set={MyMetric(), MyOtherMetric(), ...})

不带指标运行
~~~~~~~~~~~~~~~~~~~~~~~

计算风险和绩效是有开销的，它会增加回测的运行时间。在开发过程中，跳过这些指标的计算能够节省调试时间。
要跳过这些计算，用户可以将默认选择的指标集设置为 ``none`` ，例如：

.. code-block:: python

   $ zipline run algorithm.py -s 2014-01-01 -e 2014-02-01 --metrics-set none

创建新指标
~~~~~~~~~~~~~~~~~~~~

一个指标实现了下面一个或多个方法：

- ``start_of_simulation``
- ``end_of_simulation``
- ``start_of_session``
- ``end_of_session``
- ``end_of_bar``

这些函数会在需要的时候由系统调用，以收集所需要的信息。如果在函数中并不需要做任何处理，则可以省略它的定义。

指标应该被设计为可重用，即一个指标类的实例可以在多个回测中被使用。指标不需要
一次支持多个回测，只需要保证 ``start_of_simulation`` 和 ``end_of_simulation``
中的内部数据和缓存对每个回测是一致的。

``start_of_simulation``
```````````````````````

``start_of_simulation`` 可以被理解为构造函数。在函数内部应当初始化相关缓存。

``start_of_simulation`` 函数定义如下：

.. code-block:: python

   def start_of_simulation(self,
                           ledger,
                           emission_rate,
                           trading_calendar,
                           sessions,
                           benchmark_source):
       ...

``ledger`` 是 :class:`~zipline.finance.ledger.Ledger` 类的实例，它保存了回测的状态。
可以用来查询回测起始金额。

``emission_rate`` 是一个字符串类型的变量，它表示指标报告的最小频率。
``emission_rate`` 可以是 ``minute`` 或 ``daily`` 。
当 ``emission_rate`` 是 ``daily`` 时， ``end_of_bar`` 将不会被调用。

``trading_calendar`` 是 :class:`~zipline.utils.calendars.TradingCalendar` 的实例，
是回测所用的交易日历类。

``sessions`` 是 :class:`pandas.DatetimeIndex` 的实例，按执行顺序保存了会话标签。

``benchmark_source`` 是 :class:`~zipline.sources.benchmark_source.BenchmarkSource`
的实例，是 :func:`~zipline.api.set_benchmark` 设定的基准信息返回结果的接口。

``end_of_simulation``
`````````````````````

``end_of_simulation`` 的定义如下：

.. code-block:: python

   def end_of_simulation(self,
                         packet,
                         ledger,
                         trading_calendar,
                         sessions,
                         data_portal,
                         benchmark_source):
       ...

``ledger`` 是 :class:`~zipline.finance.ledger.Ledger` 类的实例，它保存了回测的状态。
可以用来查询回测起始金额。

``packet`` 是一个字典，用于写入结果数值。

``trading_calendar`` 是 :class:`~zipline.utils.calendars.TradingCalendar` 的实例，
是回测所用的交易日历类。

``sessions`` 是 :class:`pandas.DatetimeIndex` 的实例，按执行顺序保存了会话标签。

``data_portal`` 是 :class:`~zipline.data.data_portal.DataPortal` 的实例，是价格数据的接口。

``benchmark_source`` 是 :class:`~zipline.sources.benchmark_source.BenchmarkSource`
的实例，是 :func:`~zipline.api.set_benchmark` 设定的基准信息返回结果的接口。

``start_of_session``
````````````````````

如果持仓的期货价格发生了变动，或者持仓发生了变动，``start_of_session`` 和 ``end_of_session``
中的 ``ledger`` 和 ``data_portal`` 变量就会有差异。

``start_of_session`` 定义如下：

.. code-block:: python

   def start_of_session(self,
                        ledger,
                        session_label,
                        data_portal):
       ...

``ledger`` 是 :class:`~zipline.finance.ledger.Ledger` 类的实例，它保存了回测的状态。
可以用来查询回测起始金额。

``session_label`` 是 :class:`~pandas.Timestamp` 的实例，是将要运行的会话的标签。

``data_portal`` 是 :class:`~zipline.data.data_portal.DataPortal` 的实例，是价格数据的接口。

``end_of_session``
``````````````````

``end_of_session`` 定义如下：

.. code-block:: python

   def end_of_session(self,
                      packet,
                      ledger,
                      session_label,
                      session_ix,
                      data_portal):

``packet`` 是一个字典结构，用来写入回测结束后的相关数据。
这个字典又包含两个字典： ``daily_perf`` 和 ``cumulative_perf`` 。
当适用时， ``daily_perf`` 应当包含当天的数值， ``cumulative_perf`` 应当包含到目前为止的累积值。

``ledger`` 是 :class:`~zipline.finance.ledger.Ledger` 类的实例，它保存了回测的状态。
可以用来查询回测起始金额。

``session_label`` 是 :class:`~pandas.Timestamp` 的实例，是将要运行的会话的标签。

``session_ix`` 是 :class:`int` 型数据。是当前正在运行会话的下标。
通过它，可以便捷访问日收益： ``ledger.daily_returns_array[:session_ix + 1]`` 。

``data_portal`` 是 :class:`~zipline.data.data_portal.DataPortal` 的实例，是价格数据的接口。

``end_of_bar``
``````````````

.. note::

   只有当 ``emission_mode`` 是 ``minute`` ， ``end_of_bar`` 才会被调用。

``end_of_bar`` 定义如下：

.. code-block:: python

   def end_of_bar(self,
                  packet,
                  ledger,
                  dt,
                  session_ix,
                  data_portal):

``packet`` 是一个字典结构，用来写入回测结束后的相关数据。
这个字典又包含两个字典： ``daily_perf`` 和 ``cumulative_perf`` 。
当适用时， ``daily_perf`` 应当包含当天的数值， ``cumulative_perf`` 应当包含到目前为止的累积值。

``ledger`` 是 :class:`~zipline.finance.ledger.Ledger` 类的实例，它保存了回测的状态。
可以用来查询回测起始金额。

``dt`` 是 :class:`~pandas.Timestamp` 的实例，是刚刚完成的 bar 的标签。

``session_ix`` 是 :class:`int` 型数据。是当前正在运行会话的下标。
通过它，可以便捷访问日收益： ``ledger.daily_returns_array[:session_ix + 1]`` 。

``data_portal`` 是 :class:`~zipline.data.data_portal.DataPortal` 的实例，是价格数据的接口。

创建新的指标集
~~~~~~~~~~~~~~~~~~~~~~~~~

用户可以调用 :func:`zipline.finance.metrics.register` 注册新的指标集。
这个函数可以当作装饰器使用，它可以装饰一个没有参数并返回含有指标 set 结构的函数。例如：

.. code-block:: python

   from zipline.finance import metrics

   @metrics.register('my-metrics-set')
   def my_metrics_set():
       return {MyMetric(), MyOtherMetric(), ...}


这可以添加到用户的 ``extension.py`` 文件中。

指标集被定义为一个函数，而不是一个 set 结构的原因是，用户在构造过程中可能需要访问一些额外的资源数据。
将这些额外访问放到函数里，就可以在不使用指标集的时候避免这些额外的访问。
