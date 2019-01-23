.. image:: https://media.quantopian.com/logos/open_source/zipline-logo-03_.png
    :target: ./
    :width: 212px
    :align: center
    :alt: Zipline

=============

|Gitter|
|version status|
|travis status|
|appveyor status|
|Coverage Status|

Zipline是一个 Python 算法交易框架。它是一个事件驱动的回测系统。
`Quantopian <https://www.quantopian.com>`_ 是一个免费的、以社区为中心的策略编写运行平台。
Zipline 当前作为 Quantopian 的回测和实时交易引擎，正在其生产环境中运行。

- `加入社区 <https://groups.google.com/forum/#!forum/zipline>`_
- `官方英文文档 <http://www.zipline.io>`_
- 想要贡献代码? 查看 `开发指南 <./development-guidelines.html>`_

特点
========

- **易用:** Zipline 不带来额外学习负担以确保您专注于算法开发。请查看下面的代码示例。
- **类库强大:** 包含常见的指标和统计工具，如移动平均和线性回归，可以在策略中很方便地调用。
- **与 PyData 集成:** 不管是输入的价格数据，还是输出的策略评估，都是基于 Pandas 库的 DataFrames ，这很容易和 PyData 生态系统相结合。
- **统计和机器学习库:** 可以使用 matplotlib 、 scipy 、 statsmodels 、 sklearn 这些库来进行开发、分析和可视化。

安装
============

使用 ``pip`` 安装
-----------------------

假设您已经满足所有 Python 之外的依赖（见下文），您可以通过使用 ``pip`` 来安装 Zipline:

.. code-block:: bash

    $ pip install zipline

**注意:** 使用 ``pip`` 安装 Zipline 比一般的包要复杂一些。如果您从未安装过 Python
科学计算相关的包，直接运行 ``pip install zipline`` 有很大可能会失败。

复杂的原因有两个:

1. Zipline 提供了几个需要调用 CPython C API 的 C 扩展。为了构建这些 C 扩展，
``pip`` 需要访问 CPython 头文件来完成安装。

2. Zipline 依赖于 `numpy <http://www.numpy.org/>`_, 它是 Python 数值计算的核心库。而
Numpy 又依赖于 `LAPACK <http://www.netlib.org/lapack>`_ 线性代数库。

LAPACK 和 CPython 是二进制依赖关系，因此安装方法因平台而异。在 Linux 平台，用户一般通过
``apt`` 、 ``yum`` 、 ``pacman`` 这类包管理器来解决依赖问题。在 macOS
上，`Homebrew <http://www.brew.sh>`_ 是更好的选择。

获取二进制依赖关系的更多信息，请参阅完整的 `Zipline 安装文档`_ 。

conda
-----

另一种安装 Zipline 的方法是使用 ``conda`` 包管理器，它是
`Anaconda <http://continuum.io/download>`_ 的一部分，您可以通过运行
``pip install conda`` 来安装 conda 。

conda 安装好之后，就可以指定 ``Quantopian`` 频道来安装 Zipline：

.. code-block:: bash

    $ conda install -c Quantopian zipline

目前当前支持的平台有：

-  GNU/Linux 64-bit
-  macOS 64-bit
-  Windows 64-bit

.. note::

   在 Windows 32-bit 下也许可以工作，但是没有对它进行持续集成测试。

快速开始
==========

另请参阅 `入门教程 <./beginner-tutorial.html>`_.

以下代码实现了一个双移动平均的策略。

.. code:: python

    from zipline.api import order_target, record, symbol

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


可以通过 Zipline 命令行来运行这个策略，在运行之前需要申请 `Quandl <https://docs.quandl.com/docs#section-authentication>`__
的 API key 来获取默认数据。获得 key 之后，运行下面的命令：

.. code:: bash

    $ QUANDL_API_KEY=<yourkey> zipline ingest -b quandl
    $ zipline run -f dual_moving_average.py --start 2014-1-1 --end 2018-1-1 -o dma.pickle

这将从 `quandl` 下载价格数据，并对命令中指定的时间段进行回测。
结果会保存在 `dma.pickle` 文件中，您可以用 Python 加载分析。

在 ``zipline/examples`` 目录还可以找到其他例子。

有其他问题?
==========

如果您发现了 bug ，请打开 `Github issue <https://github.com/quantopian/zipline/issues/new>`_ 向我们提交。

贡献
============

欢迎提交bug 报告、bug 修复、文档改进、功能增强和任何想法。
有关如何设置开发环境的详细信息，请参阅我们的 `开发指南 <./development-guidelines.html>`_ 。

如果您打算开始使用 Zipline 代码库，可以参考 GitHub `issues` 选项卡中的问题。
有 `面向新手 <https://github.com/quantopian/zipline/issues?q=is%3Aissue+is%3Aopen+label%3A%22Beginner+Friendly%22>`_ 这类标签，
也有 `求助 <https://github.com/quantopian/zipline/issues?q=is%3Aissue+is%3Aopen+label%3A%22Help+Wanted%22>`_ 这样的标签。

也欢迎在 `邮件列表 <https://groups.google.com/forum/#!forum/zipline>`_ 或 `Gitter <https://gitter.im/quantopian/zipline>`_ 上提问。



.. |Gitter| image:: https://badges.gitter.im/Join%20Chat.svg
   :target: https://gitter.im/quantopian/zipline?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge
.. |version status| image:: https://img.shields.io/pypi/pyversions/zipline.svg
   :target: https://pypi.python.org/pypi/zipline
.. |travis status| image:: https://travis-ci.org/quantopian/zipline.png?branch=master
   :target: https://travis-ci.org/quantopian/zipline
.. |appveyor status| image:: https://ci.appveyor.com/api/projects/status/3dg18e6227dvstw6/branch/master?svg=true
   :target: https://ci.appveyor.com/project/quantopian/zipline/branch/master
.. |Coverage Status| image:: https://coveralls.io/repos/quantopian/zipline/badge.png
   :target: https://coveralls.io/r/quantopian/zipline

.. _`Zipline 安装文档` : ./install.html
