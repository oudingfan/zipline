发布流程
---------------

.. include:: dev-doc-message.txt


更新发布说明
~~~~~~~~~~~~~~~~~~~~~~~~~~

当我们准备发布新版本的 Zipline 时，会编辑 :doc:`releases` 页。
我们将在发布时维护一个新版本的 whatsnew 文件。
首先，在以下位置找到该文件： ``docs/source/whatsnew/<version>.txt`` 。
它的版本号将是当前最高版本。使用以下格式的日期格式：

::

   <month> <day>, <year>


例如： November 6, 2015。
从 whatsnew 中删除"功能正在开发"的警告，因为即将发布。
将发布的标题从“开发”更新为“发布xxx”，更新标题的下划线以匹配标题的宽度。

如果此时要重命名该版本，则需要 git mv 该文件，并更新 releases.rst 以引用重命名的文件。

要在本地构建和查看文档，请运行：

.. code-block:: bash

   $ cd docs
   $ make html
   $ {BROWSER} build/html/index.html

更新 Python stub 文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PyCharm 和其他代码类型检查器会使用 `Python stub 文件 <https://www.python.org/dev/peps/pep-0484/#stub-files>`__
用于类型提示。我们为 :mod:`~zipline.api` 生成了 stub 文件，
因为在 TradingAlgorithm 上使用的装饰器导入了它们。
因此，这些功能对静态分析工具是隐藏的，但是我们可以生成静态文件以使它们可用。
在 **Python 3** 下，运行下面命令生成 stub 文件：

.. code-block:: bash

   $ python etc/gen_type_stubs.py

.. note::

   为了使读取 stub 的工具知道所引用的类， stub 文件应该导入这些类。
   但是，因为 stub 文件中的 ``... import *`` 和 ``... import ... as ...``
   将会再次导出这些引入，所以我们显式导入名称。对于 ``zipline.api`` 的 stub 文件，
   这在上面提到的 ``gen_type_stubs.py`` 脚本的头部字符串完成。
   如果添加了新参数或返回了类型为 ``zipline.api`` 的函数，新的 import 应该被添加到该标头。

更新 ``__version__``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们使用 `versioneer <https://github.com/warner/python-versioneer>`__ 来管理
``__version__`` 和 ``setup.py`` 的版本。我们从版本控制系统的 tag 中获取版本信息，
这样可以确保版本信息在版本控制系统和代码中是同步的。

要升级版本，请使用 git tag 命令，如：

.. code-block:: bash

   $ git tag <major>.<minor>.<micro>
   $ git push && git push --tags


这将推送代码和标签信息。

接下来，在`zipline releases page <https://github.com/quantopian/zipline/releases>`__
单击 “Draft a new release” 按钮。对于新版本，选择刚刚推送的标签，然后发布该版本。

更新 PyPI 包
~~~~~~~~~~~~~~~~~~~~~~~

``sdist``
^^^^^^^^^

要构建 ``sdist`` （源代码分发），在 Zipline 的根目录运行：

.. code-block:: bash

   $ python setup.py sdist


这将创建一个包含所有 python 、 cython 和各类内容的 gzip 压缩包。
要测试 dist 是否工作正常， ``cd`` 到一个空目录，创建一个新的
virtualenv 然后运行：


.. code-block:: bash

   $ pip install <zipline-root>/dist/zipline-<major>.<minor>.<micro>.tar.gz
   $ python -c 'import zipline;print(zipline.__version__)'

这应该打印我们期望发布的新版本号。

.. note::

   ``cd`` 到一个空的目录并创建纯净的 virtualenv 环境很重要。
   更改到空目录可确保我们已包含所有清单中必需的文件。
   使用干净的 virtualenv 能确保我们已经列出所有必需的依赖包。

现在我们已经在本地测试了发布包，现在应该使用 PyPI 测试服务器进行测试了。

.. code-block:: bash

   $ pip install twine
   $ twine upload --repository-url https://test.pypi.org/legacy/ dist/zipline-<version-number>.tar.gz

Twine 将提示您输入用户名和密码，如果您被授权发布 Zipline 版本，您应该会有用户名和密码。

.. note::

   如果版本号已被使用：在本地将 setup.py 更新为一个新版本号。
   不要使用下一个版本号，只需要在当前版本附加一个 ``.<nano>`` 。
   PyPI 防止相同版本号出现两次，所以我们需要在测试服务器上调试打包时解决这个问题。

   .. warning::

      不要提交临时版本更改。

这会将 zipline 上传到 pypi 测试服务器。要测试从 pypi 安装，
创建一个新的 virtualenv ， ``cd`` 到一个空目录然后运行：

.. code-block:: bash

   $ pip install --extra-index-url https://test.pypi.org/simple zipline
   $ python -c 'import zipline;print(zipline.__version__)'


这应该下载您刚刚上传的包，然后打印版本号。

现在我们已经在本地和 PyPI 进行了测试，现在该上传到正式的 PyPI 服务器了：

.. code-block:: bash

   $ twine upload dist/zipline-<version-number>.tar.gz

``bdist``
^^^^^^^^^

因为 zipline 现在支持 numpy 多个版本，所以我们没有构建二进制 wheel ，因为没有使用特定版本。

文档
~~~~~~~~~~~~~

要更新 `zipline.io <http://www.zipline.io/index.html>`__ ，检出最新的 master 分支并运行：

.. code-block:: python

    python <zipline_root>/docs/deploy.py

这将会重新构建文档，签出 git 的 ``gh-pages`` 分支，并将构建的文档复制到 Zipline 根目录中。

.. note::

   应始终使用 **Python 3** 构建文档。许多 api 包裹的预处理函数接受 \*args 和 \**kwargs 。
   在 Python 3， sphinx 将正确处理 ``__wrapped__`` 属性并显示正确的参数。

现在，使用浏览器查看 ``index.html`` 并验证是否正确。

如果没有问题，将更新的文档推送到 GitHub 的 ``gh-pages`` 分支。

.. code-block:: bash

   $ git add .
   $ git commit -m "DOC: update zipline.io"
   $ git push origin gh-pages

`zipline.io <http://www.zipline.io/index.html>`__ 会在几分钟后更新。

更新 conda 包
~~~~~~~~~~~~~~~~~~~~~~~~

`Travis <https://travis-ci.org/quantopian/zipline>`__ 和
`AppVeyor <https://ci.appveyor.com/project/quantopian/zipline/branch/master>`__
为我们构建 zipline conda 包（分别用于 Linux / macOS 和 Windows ）。
一旦他们构建 master 的发布包并上传到anaconda.org ，
我们应该将这些包从 “ci” 标签移到 “main” 标签。
对于我们上传的所有依赖包，我们也应该这样做。您可以从这里做到这一点
`anaconda.org web界面<https://anaconda.org/Quantopian/repo>`__ 。
这也是从 anaconda 中删除所有旧 “ci” 包的好时机。

要在本地构建 conda 包，请运行：

.. code-block:: bash

   $ python etc/conda_build_matrix.py

如果所有构建成功，那么将不会打印任何内容，之后会以 ``EXIT_SUCCESS`` 退出。
如果存在构建问题，我们必须解决它。

如果所有构建都通过，我们就可以将它们上传到 anaconda ：

.. code-block:: bash

   $ python etc/conda_build_matrix.py --upload

如果您想上传给其他用户测试，可以使用 ``--user`` 指定。

下一个 Commit
~~~~~~~~~~~

为下一个版本添加 whatsnew 时，应根据次版本定义标题。
如果发布是主/次版本增量，文件可以在需要时重命名。
您可以使用 ``docs/source/whatsnew/skeleton.txt`` 作为新文件的模板。

在 ``docs/source/releases.rst`` 中包含新的 whatsnew 文件，新版本就会出现在顶部。语法是：

::

   .. include:: whatsnew/<version>.txt
