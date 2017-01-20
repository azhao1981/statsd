# Measure Anything, Measure Everything
======

If Engineering at Etsy has a religion(宗教信仰), it’s the Church(教堂) of Graphs. If it moves, we track(追踪) it. Sometimes we’ll draw a graph of something that isn’t moving yet, just in case it decides to make a run for it. In general, we tend to measure at three levels: network, machine, and application. (You can read more about our graphs in Mike’s Tracking Every Release post.)


Application metrics are usually the hardest, yet most important, of the three. They’re very specific to your business, and they change as your applications change (and Etsy changes a lot). Instead of trying to plan out(策划) everything we wanted to measure and putting it in a classical configuration management system, we decided to make it ridiculously(荒谬地) simple for any engineer to get anything they can count or time into a graph with almost no effort. (And, because we can push code anytime, anywhere, it’s easy to deploy the code too, so we can go from “how often does X happen?” to a graph of X happening in about half an hour, if we want to.)

Meet StatsD
-----------

StatsD is a simple NodeJS daemon (and by “simple” I really mean simple — NodeJS makes event-based systems like this ridiculously easy to write) that listens for messages on a UDP port. (See Flickr’s “Counting & Timing” for a previous description and implementation of this idea, and check out the open-sourced code on github to see our version.) It parses the messages, extracts(提取) metrics(度量) data, and periodically(周期性地) flushes(冲洗) the data to [graphite](http://graphiteapp.org/).


We like graphite for a number of reasons: it’s very easy to use, and has very powerful graphing and data manipulation(操纵) capabilities( 能力). We can combine data from StatsD with data from our other metrics-gathering systems. Most importantly for StatsD, you can create new metrics in graphite just by sending it data for that metric. That means there’s no management overhead for engineers to start tracking something new: simply tell StatsD you want to track “grue.dinners” and it’ll automagically appear in graphite. (By the way, because we flush data to graphite every 10 seconds, our StatsD metrics are near-realtime.)


Not only is it super easy to start capturing the rate or speed of something, but it’s very easy to view, share, and brag about them.

Why UDP?
-----------

So, why do we use UDP to send data to StatsD? Well, it’s fast — you don’t want to slow your application down in order to track its performance — but also sending a UDP packet is fire-and-forget. Either StatsD gets the data, or it doesn’t. The application doesn’t care if StatsD is up, down, or on fire; it simply trusts that things will work. If they don’t, our stats go a bit wonky(靠不住的), but the site stays up. Because we also worship(做礼拜) at the Church of Uptime(正常运行时间), this is quite alright. (The Church of Graphs makes sure we graph UDP packet receipt failures though, which the kernel usefully provides.)


Measure Anything


Here’s how we do it using our PHP StatsD library:

```php
StatsD::increment("grue.dinners");
```

That’s it. That line of code will create a new counter on the fly and increment it every time it’s executed. You can then go look at your graph and bask in(感到舒适) the awesomeness, or for that matter, spot someone up to no good in the middle of the night:

![iamge1](https://codeascraft.com/wp-content/uploads/2011/02/logins2.png)

We can use graphite’s data-processing tools to take the the data above and make a graph that highlights deviations(偏差) from the norm(标准):

![iamge2](https://codeascraft.com/wp-content/uploads/2011/02/login_fails2.png)

(We sometimes use the “rawData=true” option in graphite to get a stream of numbers that can feed into automatic monitoring systems. Graphs like this are very “monitorable(可监测).”)


We don’t just track trivial(不重要的) things like how many people are signing(签字) into the site — we also track really important stuff, like how much coffee is left in(放在) the kitchen(厨房):

![images3](https://codeascraft.com/wp-content/uploads/2011/02/coffee1.png)


Time Anything Too
--------

In addition to(除…以外) plain counters, we can track times too:

```php
$start = microtime(true);
eat_adventurer();
StatsD::timing("grue.dinners", (microtime(true) - $start) * 1000);
```

StatsD automatically tracks the count, mean(平均), maximum, minimum, and 90th percentile(百分率) times (which is a good measure of “normal” maximum values, ignoring outliers(离群值)). Here, we’re measuring the execution(执行) times of part of our search infrastructure(基础设施):

![images](https://codeascraft.com/wp-content/uploads/2011/02/timing1.png)

Sampling Your Data
-----

One thing we found early on is that if we want to track something that happens really, really frequently, we can start to overwhelm(淹没) StatsD with UDP packets. To cope(处理) with that, we added the option to sample data, i.e. to only send packets a certain percentage(百分比) of the time. For very frequent events, this still gives you a statistically(统计地) accurate view of activity(活动).

To record only one in ten events:

```php
StatsD::increment(“adventurer.heartbeat”, 0.1);
```

What’s important here is that the packet sent to StatsD includes the sample rate, and so StatsD then multiplies(乘) the numbers to give an estimate(估计) of a 100% sample rate before it sends the data on to graphite. This means we can adjust the sample rate at will without having to deal with rescaling(尺度改变) the y-axis of the resulting graph.


Measure Everything
-----

We’ve found that tracking everything is key to moving fast, but the only way to do it is to make tracking anything easy. Using StatsD, we enable engineers to track what they need to track, at the drop of a hat, without requiring time-sucking configuration changes or complicated processes.

Try StatsD for yourself: grab(攫取) the open-sourced code from github and start measuring. We’d love to hear what you think of it.











