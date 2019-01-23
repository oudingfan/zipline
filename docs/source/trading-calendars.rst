交易日历
-----------------

什么是交易日历？
~~~~~~~~~~~~~~~~~~~~~~~~~~~
交易日历表示交易所的时间信息。
时间信息由两部分组成：会话和开市闭市时间。这由Zipline的 :class:`~zipline.utils.calendars.trading_calendar.TradingCalendar` 类表示，并用作于所有 ``TradingCalendar``  的实例。

会话由一组连续的分钟组成，并具有一个 UTC 时区零点时刻的标签。需要注意的是，标签不应被视为特定时间，UTC 时区零点时刻仅用于方便。

对于一个 `纽约证券交易所 <https://www.nyse.com/index>`__ 的一般交易日来说，市场开放时间为上午9点30分至下午4点。交易时段可能会根据交易所或特殊日子而调整。


为什么需要关心日历交易？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

假设您想在周二购入股票，然后在周六卖出。如果交易所在周六没有开放，实际上那时无法卖出，必须等到下一个交易日进行交易。由于现实生活中无法在周六进行交易，因此在回测中这样做也是不合理的。

为了让回测准确， `数据 bundle <http://www.zipline.io/bundles.html>`__ 中的日期和 ``TradingCalendar`` 中的日期应该匹配；如果不匹配，将会报错。这对分钟级和日线级数据同样适用。


TradingCalendar 类
~~~~~~~~~~~~~~~~~~~~~~~~~

如果我们要构建我们自己的 ``TradingCalendar``  类，其中有许多我们需要考虑的属性，包括：

  - 交易所名称
  - 时区
  - 开市时间
  - 闭市时间
  - 固定和临时假期
  - 临时的开市和闭市时间

如果您想查看 ``TradingCalendar`` 所有的可用属性和方法，请查看 `API Reference <http://www.zipline.io/appendix.html#trading-calendar-api>`__ 。

现在来看看伦敦证券交易所日历类： :class:`~zipline.utils.calendars.exchange_calendar_lse.LSEExchangeCalendar` ，如下例所示：

.. code-block:: python

  class LSEExchangeCalendar(TradingCalendar):
    """
    Exchange calendar for the London Stock Exchange

    Open Time: 8:00 AM, GMT
    Close Time: 4:30 PM, GMT

    Regularly-Observed Holidays:
      - New Years Day (observed on first business day on/after)
      - Good Friday
      - Easter Monday
      - Early May Bank Holiday (first Monday in May)
      - Spring Bank Holiday (last Monday in May)
      - Summer Bank Holiday (last Monday in May)
      - Christmas Day
      - Dec. 27th (if Christmas is on a weekend)
      - Boxing Day
      - Dec. 28th (if Boxing Day is on a weekend)
    """

    @property
    def name(self):
      return "LSE"

    @property
    def tz(self):
      return timezone('Europe/London')

    @property
    def open_time(self):
      return time(8, 1)

    @property
    def close_time(self):
      return time(16, 30)

    @property
    def regular_holidays(self):
      return HolidayCalendar([
        LSENewYearsDay,
        GoodFriday,
        EasterMonday,
        MayBank,
        SpringBank,
        SummerBank,
        Christmas,
        WeekendChristmas,
        BoxingDay,
        WeekendBoxingDay
      ])


您可以通过 `pandas <http://pandas.pydata.org/pandas-docs/stable/>`__  ，来创建 ``def regular_holidays(self)`` 中使用到的 ``Holiday`` 模块， ``pandas.tseries.holiday.Holiday`` ，也可以查看 `LSEExchangeCalendar <https://github.com/quantopian/zipline/blob/master/zipline/utils/calendars/exchange_calendar_lse.py>`__ 示例，或者参考下面的代码示例。

.. code-block:: python

  from pandas.tseries.holiday import (
      Holiday,
      DateOffset,
      MO
  )

  SomeSpecialDay = Holiday(
      "Some Special Day",
      month=1,
      day=9,
      offset=DateOffSet(weekday=MO(-1))
  )


编写自定义交易日历
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

现在来构建自定义交易日历。此日历将用于 7 * 24 小时交易，
即它将在周一至周日开放，时间段为上午12点至晚上11点59分。
使用的时区为 UTC 时区。

首先引入一些需要的模块：

.. code-block:: python

  # for setting our open and close times
  from datetime import time
  # for setting our start and end sessions
  import pandas as pd
  # for setting which days of the week we trade on
  from pandas.tseries.offsets import CustomBusinessDay
  # for setting our timezone
  from pytz import timezone

  # for creating and registering our calendar
  from trading_calendars import register_calendar, TradingCalendar
  from zipline.utils.memoize import lazyval


现在我们创建了一个名为 ``TFSExchangeCalendar`` 的类：

.. code-block:: python

  class TFSExchangeCalendar(TradingCalendar):
    """
    An exchange calendar for trading assets 24/7.

    Open Time: 12AM, UTC
    Close Time: 11:59PM, UTC
    """

    @property
    def name(self):
      """
      The name of the exchange, which Zipline will look for
      when we run our algorithm and pass TFS to
      the --trading-calendar CLI flag.
      """
      return "TFS"

    @property
    def tz(self):
      """
      The timezone in which we'll be running our algorithm.
      """
      return timezone("UTC")

    @property
    def open_time(self):
      """
      The time in which our exchange will open each day.
      """
      return time(0, 0)

    @property
    def close_time(self):
      """
      The time in which our exchange will close each day.
      """
      return time(23, 59)

    @lazyval
    def day(self):
      """
      The days on which our exchange will be open.
      """
      weekmask = "Mon Tue Wed Thu Fri Sat Sun"
      return CustomBusinessDay(
        weekmask=weekmask
      )


结论
~~~~~~~~~~~

为了让策略算法能使用这个日历类，需要一个包含所有交易日数据的  bundle 。您可以在 `Writing a New Bundle <http://www.zipline.io/bundles.html#writing-a-new-bundle>`__ 文档中阅读有关如何创建自己的数据包的信息，或者使用 `csvdir bundle <https://github.com/quantopian/zipline/blob/master/zipline/data/bundles/csvdir.py>`__ 从CSV文件创建 bundle .
