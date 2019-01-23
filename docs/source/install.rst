安装
=======

使用 ``pip`` 安装
-----------------------

使用 ``pip`` 安装 Zipline 比一般的包要复杂一些。

复杂的原因有两个:

1. Zipline 提供了几个需要调用 CPython C API 的 C 扩展。为了构建这些 C 扩展，
``pip`` 需要访问 CPython 头文件来完成安装。

2. Zipline 依赖于 `numpy <http://www.numpy.org/>`_, 它是 Python 数值计算的核心库。而
Numpy 又依赖于 `LAPACK <http://www.netlib.org/lapack>`_ 线性代数库。

LAPACK 和 CPython 是二进制依赖关系，因此安装方法因平台而异。
如果您更愿意使用单个工具来安装 Python 和非 Python 依赖，或者已将
`Anaconda <http://continuum.io/downloads>`_ 作为 Python 管理器，您可以跳至
:ref:`使用 Conda 安装 <conda>`。

安装了必要的附加依赖（请参阅下面的特定平台）之后，您应该能够运行下面语句进行安装：

.. code-block:: bash

   $ pip install zipline

如果您还使用 Python 做其他事情， **强烈** 建议您在
`virtualenv <https://virtualenv.readthedocs.org/en/latest>`_ 中安装。
`Hitchhiker’s Guide to Python`_ 提供了 `优秀的 virtualenv 教程
<http://docs.python-guide.org/en/latest/dev/virtualenvs/>`_ 。

GNU/Linux
~~~~~~~~~

在 `Debian衍生`_ Linux 发行版中, 您可以使用 ``apt`` 一次性安装所有依赖：

.. code-block:: bash

   $ sudo apt-get install libatlas-base-dev python-dev gfortran pkg-config libfreetype6-dev

在最新的 `RHEL衍生`_ Linux 发行版 （如 Fedora）中, 下面的命令也可以安装所有依赖：

.. code-block:: bash

   $ sudo dnf install atlas-devel gcc-c++ gcc-gfortran libgfortran python-devel redhat-rep-config

在 `Arch Linux`_ 中，您可以使用 ``pacman`` 安装依赖:

.. code-block:: bash

   $ pacman -S lapack gcc gcc-fortran pkg-config

还有一些 AUR 软件包可用于安装 `Python 3.4 <https://aur.archlinux.org/packages/python34/>`_
（Arch 的默认 Python 现在是 3.5，但 Zipline 目前只支持到 3.4）和
`ta-lib <https://aur.archlinux.org/packages/ta-lib/>`_ （Zipline 的可选依赖）。
Python 2 可以这样安装:

.. code-block:: bash

   $ pacman -S python2

macOS
~~~~~

macOS 附带的 Python 版本是老版本的，并且由于它还被操作系统使用，使用起来颇为不便。
所以，许多开发人员选择单独安装和使用 Python 。
`Hitchhiker’s Guide to Python`_ 的指南中有一篇不错的文章
`Installing Python on OSX <http://docs.python-guide.org/en/latest/>`_ ，
解释了如何使用 `Homebrew`_ 安装独立的 Python 。

假设您已经用 Homebrew 安装了 Python ，您可以这样安装依赖：

.. code-block:: bash

   $ brew install freetype pkg-config gcc openssl

Windows
~~~~~~~

在 Windows 下，最容易也是最好的方法是使用 :ref:`Conda <conda>` 。

.. _conda:

使用 ``conda`` 安装
-------------------------

另一种安装 Zipline 的方法是使用 conda 包管理器，它是
`Anaconda <http://continuum.io/downloads>`_ 发行版的一部分。

在 Conda 中使用 pip 的主要优点是，conda 本身可以理解 numpy 和 scipy 这些包的依赖性。
这意味着只使用 conda 一个工具，就可以安装 Zipline 及其依赖项（包括 Python 包以外的依赖）。

有关如何安装 ``conda`` ，请参阅
`Conda Installation Documentation <http://conda.pydata.org/docs/download.html>`_

``conda`` 安装好之后，就可以指定 ``Quantopian`` 频道来安装 Zipline：

.. code-block:: bash

    conda install -c Quantopian zipline

.. _`Debian衍生`: https://www.debian.org/misc/children-distros
.. _`RHEL衍生`: https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux_derivatives
.. _`Arch Linux` : https://www.archlinux.org/
.. _`Hitchhiker’s Guide to Python` : http://docs.python-guide.org/en/latest/
.. _`Homebrew` : http://brew.sh

管理 ``conda`` 环境
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
推荐使用 ``conda`` 隔离环境来安装 Zipline 。
在 ``conda`` 环境中安装 Zipline 可以不影响您默认的 Python 环境或 site-packages，
这能够防止在全局环境中发生冲突。
有关 ``conda`` 环境的更多信息，请参阅
`Conda User Guide <https://conda.io/docs/user-guide/tasks/manage-environments.html>`_ 。

假设 ``conda`` 已经设置好，现在可以创建一个 ``conda`` 环境：

- Python 2.7:

.. code-block:: bash

    $ conda create -n env_zipline python=2.7

- Python 3.5:

.. code-block:: bash

    $ conda create -n env_zipline python=3.5


现在您已经创建了一个名为 ``env_zipline`` 的沙盒隔离环境来安装 Zipline 。
然后您可以使用下面的命令来激活conda环境：

.. code-block:: bash

    $ source activate env_zipline

您可以通过下面命令来安装 Zipline ：

.. code-block:: bash

    (env_zipline) $ conda install -c Quantopian zipline

停用 ``conda`` 环境：

.. code-block:: bash

    (env_zipline) $ source deactivate

