---
layout: post
title: "YouTube Music Data Consumption - Vnstat"
comments: true
keywords: "YouTube, YouTube Music, Data Consumption, Vnstat, Throttling"
---

YouTube Music doesn't provide any option to control video quality. There exists a feature which help you change the streaming quality of audio, but I think it's useless. Moreover, the video quality is gets automatically adjusted depending upon your connection speed.

With the help of [`vnstat`](https://github.com/vergoh/vnstat), I monitored the data usage of [ YouTube Music's](music.youtube.com) initial load in Chrome's Guest Session.

```sh
$ vnstat -l -i enp0s20u1u4
```

`enp0s20u1u4` is my default interface.

```sh
                           rx         |       tx
--------------------------------------+------------------
  bytes                     4.32 MiB  |         382 KiB
--------------------------------------+------------------
          max            1.76 Mbit/s  |      178 kbit/s
      average          589.62 kbit/s  |    50.91 kbit/s
          min               0 kbit/s  |        0 kbit/s
--------------------------------------+------------------
  packets                       3623  |            2438
--------------------------------------+------------------
          max                161 p/s  |         161 p/s
      average                 60 p/s  |          40 p/s
          min                  0 p/s  |           0 p/s
--------------------------------------+------------------
  time                    60 seconds

```

- `rx` ~ *downloaded* or *received* __and__ `tx` ~ *uploaded* or *transmitted*

4.32 MiB of data was utilized during the fresh load of YouTube Music, which gets reduced to __151 KiB__ on another reload. Cache!


__Important__:

<p> 
  <li>A mebibyte (MiB) equals \(2^{20}\) bytes.</li>
  <li>To get the value in MB, all we have to do is (MiB x 1.049) </li>
  <li>4.32 MiB = ~4.52 MB</li>
  <li>For the sake of simplicity, I'll stick to MiB</li>
</p>

```sh
                           rx         |       tx
--------------------------------------+------------------
  bytes                      151 KiB  |          77 KiB
--------------------------------------+------------------
          max             190 kbit/s  |       94 kbit/s
      average           46.57 kbit/s  |    23.74 kbit/s
          min               0 kbit/s  |        0 kbit/s
--------------------------------------+------------------
```

So far, everything seem pretty useless, but I just want to check two things - 

- Whether reducing audio quality helps in less consumption of data. I had already said that the feature is useless, but I'm not quite sure about it.

- Playing with [Bandwidth Throttling](https://en.wikipedia.org/wiki/Bandwidth_throttling) to reduce video quality so that I can check the difference of data usage while streaming the same song over different network speed.

<br>

#### Difference in Data Consumption - Audio Quality Change

Song - [Geom - Back To You](https://music.youtube.com/watch?v=O9PEK5CIzg4&feature=share). Throughout the whole post, I'll use the same song.

I usually listen to it while working. Try not to watch it when you're doing something, it's kind of distracting.


Running `vnstat` for checking live transfer rate gives - 

```sh
                           rx         |       tx
--------------------------------------+------------------
  bytes                    15.94 MiB  |         450 KiB
--------------------------------------+------------------
          max            4.53 Mbit/s  |      103 kbit/s
      average          483.72 kbit/s  |    13.34 kbit/s
          min               0 kbit/s  |        0 kbit/s
--------------------------------------+------------------
  packets                      12794  |            2801
--------------------------------------+------------------
          max                423 p/s  |          69 p/s
      average                 47 p/s  |          10 p/s
          min                  0 p/s  |           0 p/s
--------------------------------------+------------------
  time                  4.50 minutes
```

Song's actual length is __4.24 min__, but here it shows __4.50 min__ and that's because I forgot to play the song. It didn't made any difference in data consumption.

__So far, I've been doing everything using Chrome's Guess Session, but looks like YouTube Music doesn't let you change audio quality if you aren't logged in. Alright, I'll just log in.__

After changing the __audio quality__, here's what I got - 

```sh
                           rx         |       tx
--------------------------------------+------------------
  bytes                    13.55 MiB  |         350 KiB
--------------------------------------+------------------
          max            4.46 Mbit/s  |      109 kbit/s
      average          408.09 kbit/s  |    10.28 kbit/s
          min               0 kbit/s  |        0 kbit/s
--------------------------------------+------------------
```

Before, it was __15.94 MiB__ and now it's __13.55 MiB__. So, I saved __2.4 MiB__ of data. I think it won't make a big of a difference.

__Conclusion__ : Changing audio quality isn't an efficient solution to reduce data usage while streaming music.

<br>

#### Experimenting with Bandwidth Throttling

Chrome's latest version provides three options to manipulate connection speed. Won't dig too much. Just a quick check over different network options.

- __Online__

    - This took __15.94 MiB__ of data to play a __4.24 min__ long song.

    - My internet connection speed was __19 Mbps__.

- __Fast 3G__

    - I didn't see any change. Quite disappointing, but I had already guessed the results.

    - Consumed __15.80 MiB__ of data.

- __Slow 3G__

    - Lagged, a couple of times.

    - Video quality was reduced.

    - Consumed `bytes: 7.65 MiB (rx) | 282 KiB (tx)`. Reasonable amount of data consumption if you just want to listen songs while doing your work.

    - Saved __`8.15 MiB`__

    - Almost half of the data used in comparison to __Normal 4G__ and __Fast 3G__ network.

- __Custom__
    
    - I want to test how does YouTube Music performs over 2G networks, and in order to do that, I need to set values for upload, download, and latency using Chrome's throttling feature.
    
    - I'm using [2G, 3G, 4G, and 5G Download Comparison chart](https://kenstechtips.com/index.php/download-speeds-2g-3g-and-4g-actual-meaning) as the source to set the __download/upload speed__ of __2G network__.

    ```
    Generation: 2G
    Technology: EDGE
    Maximum Download Speed: 0.3Mbit/s or 300 Kbps
    Maximum Upload Speed: 0.1Mbit/s or 100 Kbps
    ```
__NOTE__ - Latency is inversely proportional to the network efficiency.

For 2G Network, we can set the latency to __130 ms__ (the range is 100-500 ms).

```sh
                           rx         |       tx
--------------------------------------+------------------
         bytes              6.76 MiB  |         349 KiB
--------------------------------------+------------------
          max            2.36 Mbit/s  |      408 kbit/s
      average          200.55 kbit/s  |    10.13 kbit/s
          min               0 kbit/s  |        0 kbit/s
--------------------------------------+------------------
          time                        |    4.30 minutes
```

__Interesting!__ Switching from __Fast 3G__ to __2G EDGE__ didn't put much of the impact as the. The data consumption got reduced by __~13.16%__. I was expecting much more. 

For the final test, I'll __reduce 50%__ of download/upload speed and __increase 50%__ of latency to see what's the result.

- Download Speed: __150 Kbps__
- Upload Speed: __50 Kbps__
- Latency: __195 ms__

This is what I got -

```sh
                           rx         |       tx
--------------------------------------+------------------
  bytes                     5.82 MiB  |         302 KiB
--------------------------------------+------------------
          max             716 kbit/s  |       59 kbit/s
      average          147.08 kbit/s  |     7.45 kbit/s
          min               0 kbit/s  |        0 kbit/s
--------------------------------------+------------------
```

__Awesome! The data usage went down from `15.94 MiB` which got utilized over 4G network to `5.82 MiB` (actually slower than 2G EDGE).__

I didn't compromise with the audio quality. There was no reason to. 


Yes, there were multiple lags while streaming the video, but it was expected. I set up the upload/download way too low. I think with a little more tweaking with the throttling, I can manage to get a decent speed so that not only I can control the useless data usage, but stream music without any lags.

***

 __Links__

- [What does Fast 3G actually mean?](https://stackoverflow.com/questions/48833626/what-does-fast-3g-actually-mean)
- __[Latency - Definition](http://www.linfo.org/latency.html)__
- __[How Latency Can Make Even Fast Internet Connections Feel Slow](https://www.howtogeek.com/138771/htg-explains-how-latency-can-make-even-fast-internet-connections-feel-slow/)__
- [LTE Latency](https://www.cablefree.net/wirelesstechnology/4glte/lte-network-latency/)
- [Troubleshooting Latency Using Traceroute](https://support.sugarcrm.com/Knowledge_Base/Troubleshooting/Troubleshooting_Latency_Using_Traceroute/)
- [Understanding Low Bandwidth and High Latency](https://developers.google.com/web/fundamentals/performance/poor-connectivity)
- [MiB - mebibyte](https://searchstorage.techtarget.com/definition/mebibyte-MiB)
- [Mebibyte - Wikipedia](https://simple.wikipedia.org/wiki/Mebibyte)
- [Megabits vs Megabytes](https://volo.net/faq/whats-the-difference-between-megabits-and-megabytes)
- [Calculate Download Time](https://www.download-time.com/)
- [Downloading Speed: What do 2G, 3G, 4G, and 5G actually mean?](https://kenstechtips.com/index.php/download-speeds-2g-3g-and-4g-actual-meaning)
- [Why do mobile networks have high latency, and how it can be reduced?](https://serverfault.com/questions/387627/why-do-mobile-networks-have-high-latencies-how-can-they-be-reduced)
- [What is network latency and how do you use a latency calculator to calculate throughput?](https://www.sas.co.uk/blog/what-is-network-latency-how-do-you-use-a-latency-calculator-to-calculate-throughput)