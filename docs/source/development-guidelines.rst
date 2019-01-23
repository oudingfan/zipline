开发指南
======================
此页面适用于 Zipline 的开发人员，或想要为 Zipline 代码或文档做出贡献的人员，或想要从源代码安装并对本地副本进行更改的人员。

欢迎所有 bug 报告、 bug 修复、文档改进、建议和想法。我们在 `GitHub`__ 上 `跟踪 issue`__ ，同时也有 `邮件列表`__ 可以提问问题。

__ https://github.com/quantopian/zipline/issues
__ https://github.com/
__ https://groups.google.com/forum/#!forum/zipline

创建开发环境
----------------------------------

首先，您需要 clone Zipline ：

.. code-block:: bash

   $ git clone git@github.com:your-github-username/zipline.git

然后新建分支，您可以在其中进行更改：

.. code-block:: bash

   $ git checkout -b some-short-descriptive-name

如果您还没有安装 C 库依赖项，您可以查看 `安装指南`__ 获取相应的依赖项。

__ install.html

以下部分假设您已在系统上安装了 virtualenvwrapper 和 pip 。然后安装用于开发的 Python 依赖库：

.. code-block:: bash

   $ mkvirtualenv zipline
   $ ./etc/ordered_pip.sh ./etc/requirements.txt
   $ pip install -r ./etc/requirements_dev.txt
   $ pip install -r ./etc/requirements_blaze.txt

最后，您可以通过运行以下命令来构建 C 扩展：

.. code-block:: bash

   $ python setup.py build_ext --inplace

如果你想在本地安装软件包：

.. code-block:: bash

   $ pip install -e .
   $ zipline --help

结束之前确保所有 tests 运行通过。

如果在设置了新的 virtualenv 后运行 nosetests 时出错，请尝试运行：

.. code-block:: bash

   # zipline 是您 virtualenv 的名字
   $ deactivate zipline
   $ workon zipline


使用 Docker 开发
-----------------------

如果您想在 `Docker`__ 容器中使用 Zipline ，您需要在 Zipline 的根目录中创建 ``Dockerfile`` ，然后创建 ``Dockerfile-dev `` 。
有关构建两个容器的说明可分别在 ``Dockerfile`` 和 ``Dockerfile-dev`` 中找到。

__ https://docs.docker.com/get-started/


代码风格 & 测试
---------------------------

我们使用 `flake8`__ 来检查样式，使用 `nosetests`__ 来运行 Zipline 测试。我们的 `持续集成`__ 工具将运行这些命令。

__ http://flake8.pycqa.org/en/latest/
__ http://nose.readthedocs.io/en/latest/
__ https://en.wikipedia.org/wiki/Continuous_integration

在提交补丁或提取请求之前，请确保通过了测试：

.. code-block:: bash

   $ flake8 zipline tests

为了在本地运行测试，你需要 `TA-lib`__ ，你可以在 Linux 上安装它：

__ https://mrjbq7.github.io/ta-lib/install.html

.. code-block:: bash

   $ wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz
   $ tar -xvzf ta-lib-0.4.0-src.tar.gz
   $ cd ta-lib/
   $ ./configure --prefix=/usr
   $ make
   $ sudo make install

在 macOS ，你可以运行：

.. code-block:: bash

   $ brew install ta-lib

然后运行 ``pip install`` TA-lib ：

.. code-block:: bash

   $ pip install -r ./etc/requirements_talib.txt

您现在可以运行测试了：

.. code-block:: bash

   $ nosetests


持续集成
----------------------

我们使用 `Travis CI`__ 进行 Linux-64 构建，使用 `AppVeyor`__ 进行 Windows-64 构建。

.. note::

   我们目前没有 macOS-64 构建的 CI 。32位版本可能运行成功但不包含在我们的集成测试中。

__ https://travis-ci.org/quantopian/zipline
__ https://ci.appveyor.com/project/quantopian/zipline


打包
---------
要了解我们如何构建 Zipline conda 包，您可以在我们的发布流程说明中阅读 `这里`__ 。

__ release-process.html#uploading-conda-packages

文档贡献
------------------------

如果您想贡献 zipline.io 的文档，可以查看 ``docs/source/`` 目录，其中每个 `reStructuredText`__ （``.rst``） 文件都代表一个章节。要添加一个章节，请创建一个名为 ``some-descriptive-name.rst`` 的文件，并将 ``some-descriptive-name`` 添加到 ``appendix.rst`` 。要编辑一个章节，只需打开一个现有文件，更改并保存。

__ https://en.wikipedia.org/wiki/ReStructuredText

我们使用 `Sphinx`__ 为 Zipline 生成文档，您可以这样安装：

__ http://www.sphinx-doc.org/en/stable/


.. code-block:: bash

   $ pip install -r ./etc/requirements_docs.txt

要在本地构建和查看文档，请运行：

.. code-block:: bash

   # 假设您已经在 Zipline 根目录
   $ cd docs
   $ make html
   $ {BROWSER} build/html/index.html


版本提交信息
---------------

版本提交信息的前缀：

.. code-block:: text

   BLD：与构建 Zipline相 关的更改
   BUG：bug 修复
   DEP：已弃用，或删除已弃用的对象
   DEV：开发工具或实用程序
   DOC：文档
   ENH：功能增强
   MAINT：维护提交（重构，拼写错误等）
   REV：恢复先前的提交
   STY：代码样式修改（空格、 PEP8 、 flake8 等）
   TST：添加或修改测试
   REL：与 Zipline 版本发布有关
   PERF：性能增强


一些提交样式指南：

提交行不应超过 `72个字符`__ 。提交的第一行应包括上述前缀之一。主题和正文之间应该有一个空行。一般来说，信息应该是祈使语句。最好不仅包括变更内容，还包括变更原因。

__ https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project

**示例：**

.. code-block:: text

   MAINT: Remove unused calculations of max_leverage, et al.

   In the performance period the max_leverage, max_capital_used,
   cumulative_capital_used were calculated but not used.

   At least one of those calculations, max_leverage, was causing a
   divide by zero error.

   Instead of papering over that error, the entire calculation was
   a bit suspect so removing, with possibility of adding it back in
   later with handling the case (or raising appropriate errors) when
   the algorithm has little cash on hand.


格式化 Docstring
---------------------

在为类和函数等编写 Docstring 时，我们参考 `numpy`__ 的规范。

__ https://github.com/numpy/numpy/blob/master/doc/HOWTO_DOCUMENT.rst.txt


更新 Whatsnew
---------------------

我们有一些 `whatsnew <https://github.com/quantopian/zipline/tree/master/docs/source/whatsnew>`__ 文件，用于记录不同版本发生的变化。
您对 Zipline 更改之后，请在您的 Pull Request 中更新最新的 ``whatsnew`` 文件，并附上评论。您可以查看之前的 ``whatsnew`` 文件示例。
