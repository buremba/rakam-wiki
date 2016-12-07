## Rakam vs Google Analytics

Google Analytics is the leading advanced analytics solution mainly for websites. It provides tons of features 
from behavioral analytics to real-time analytics and the analytics service providers learned too much from Google Analytics over the years, Google shares its analytics know-how with Google Analytics to outside world.
However similar to other Google products, Google Analytics has its own world. You should learn how to use Google Analytics 
and probably get help from Google Analytics experts in order to be able to use it efficiently.

The main problem with Google Analytics is that you don't know how Google defined the metrics. 
Let's take a look at *time_on_page* metric in Google Analytics: Interestingly when you install both Google Analytics 
and some other analytics service such as Amplitude, you will see that the value of the same metric *time_on_page* is quite different.
That's because this metric has different definition among the analytics services and the services has different trade-offs calculating the value of this metric.
If you have a blog or an e-commerce website, the definition of *time_on_page* should differ because the mechanics of the visitors are alike but you can't change the behavior of these services.

The other problem is that Google Analytics has a set of pre-defined reports and 
just gives you the ability to change the parameters in pre-defined reports. You can't create new kind of reports and may hit limitations 
when you want to analyze the data specific to your company because Google Analytics is a generic solution for all websites. Unless you are a Google 360 Suite customer, 
you can't export the raw data to Google BigQuery and get stuck with the free version of Google Analytics.

The last concern about Google Analytics is the sampling problem. If you have more than a few millions of events monthly or drill-down the events with different dimensions, 
Google Analytics will [sample your data](https://en.wikipedia.org/wiki/Sampling_(statistics)) so you can't trust the metrics you see on Google Analytics.
It may seem not that important for you but *it actually is*. For example one of our e-commerce customers with ~400K events daily were using free version of Google Analytics and 
when they installed Rakam, the metrics in Google Analytics were 2x of what they see on Rakam just because the sampling method they use.
If you become a paid customer and switch to Google 360 Suite, Google Analytics will not sample your data but you may want to take a look at the price of that product.
