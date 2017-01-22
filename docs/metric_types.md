StatsD Metric Types
==================


Counting
--------

    gorets:1|c

This is a simple counter. Add 1 to the "gorets" bucket.
At each flush the current count is sent and reset to 0.
If the count at flush is 0 then you can opt to send no metric at all for
this counter, by setting `config.deleteCounters` (applies only to graphite
backend).  Statsd will send both the rate as well as the count at each flush.

### Sampling

    gorets:1|c|@0.1

Tells StatsD that this counter is being sent sampled every 1/10th of the time.

Timing
------

    glork:320|ms|@0.1

The glork took 320ms to complete this time. StatsD figures out(算出) percentiles(百分比),
average (mean), standard deviation, sum, lower and upper bounds(最大值) for the flush interval(每个刷新间隔的).
The percentile threshold can be tweaked(调整) with `config.percentThreshold`.

The percentile threshold can be a single value, or a list of values, and will
generate the following list of stats for each threshold:

    stats.timers.$KEY.mean_$PCT
    stats.timers.$KEY.upper_$PCT
    stats.timers.$KEY.sum_$PCT

Where `$KEY` is the stats key you specify when sending to statsd, and `$PCT` is
the percentile threshold.

Note that the `mean` metric is the mean value of all timings recorded during
the flush interval whereas(反之) `mean_$PCT` is the mean of all timings which fell
into(变成) the `$PCT` percentile for that flush interval. And the same holds(同样) for sum
and upper. See [issue #157](https://github.com/etsy/statsd/issues/157) for a
more detailed explanation of the calculation.

If the count at flush is 0 then you can opt to send no metric at all for this timer,
by setting `config.deleteTimers`.

Use the `config.histogram` (直方图) setting to instruct(命令) statsd to maintain(维持) histograms
over time(随着时间的过去).  Specify which metrics to match and a corresponding list of
ordered non-inclusive upper limits of bins(工具屉) (class intervals(间隔)).
(use `inf` to denote(指示) infinity(无限大); a lower limit of 0 is assumed(假定的))
Each `flushInterval`, statsd will store how many values (absolute frequency)
fall within each bin (class interval), for all matching metrics.
Examples:

* no histograms for any timer (default): `[]`
* histogram to only track render durations,
  with unequal class intervals and catchall(装杂物的容器) for outliers(离群值):

        [ { metric: 'render', bins: [ 0.01, 0.1, 1, 10, 'inf'] } ]

* histogram for all timers except 'foo' related,
  with equal class interval and catchall for outliers:

        [ { metric: 'foo', bins: [] },
          { metric: '', bins: [ 50, 100, 150, 200, 'inf'] } ]

Statsd also maintains a counter for each timer metric. The 3rd field
specifies the sample rate(抽样率) for this counter (in this example @0.1). The field
is optional and defaults to 1.

Note:

* first match for a metric wins.
* bin upper limits(上限) may contain decimals(小数).
* this is actually more powerful than what's strictly(严格地) considered
histograms, as you can make each bin arbitrarily(武断地) wide(大),
i.e. class intervals of different sizes.

Gauges(计量表)
------
StatsD now also supports gauges, arbitrary(任意) values, which can be recorded.

    gaugor:333|g

If the gauge is not updated at the next flush, it will send the previous value. You can opt to send
no metric at all for this gauge, by setting `config.deleteGauges`

Adding a sign to the gauge value will change the value, rather than setting it.

    gaugor:-10|g
    gaugor:+4|g

So if `gaugor` was `333`, those commands would set it to `333 - 10 + 4`, or
`327`.

Note:

This implies you can't explicitly(明确地) set a gauge to a negative number
without first setting it to zero.

Sets
----
StatsD supports counting unique occurences of events between flushes,
using a Set to store all occuring events.

    uniques:765|s

If the count at flush is 0 then you can opt to send no metric at all for this set, by
setting `config.deleteSets`.

Multi-Metric Packets
--------------------
StatsD supports receiving multiple metrics in a single packet by separating them
with a newline.

    gorets:1|c\nglork:320|ms\ngaugor:333|g\nuniques:765|s

Be careful to keep the total length of the payload within your network's MTU. There
is no single good value to use, but here are some guidelines for common network
scenarios:

* Fast Ethernet (1432) - This is most likely for Intranets.
* Gigabit Ethernet (8932) - Jumbo frames can make use of this feature much more
  efficient.
* Commodity Internet (512) - If you are routing over the internet a value in this
  range will be reasonable. You might be able to go higher, but you are at the mercy
  of all the hops in your route.

*(These payload numbers take into account the maximum IP + UDP header sizes)*


