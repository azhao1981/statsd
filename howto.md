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

```
StatsD::increment("grue.dinners");
```

That’s it. That line of code will create a new counter on the fly and increment it every time it’s executed. You can then go look at your graph and bask in(感到舒适) the awesomeness, or for that matter, spot someone up to no good in the middle of the night:

[iamge1](https://codeascraft.com/wp-content/uploads/2011/02/logins2.png)



