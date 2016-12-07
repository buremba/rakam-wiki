## Rakam vs Tableau and BI solutions

Occasionally we get questions about the similarity between Rakam and Tableau or other BI solutions, 
so I wanted to add a comparison page for BI solutions. To make it clear, Rakam is not a BI product, instead a full-stack analytics platform.
There are two products under the Rakam platform: Rakam API and Rakam UI. 

Rakam API allows you collect data from multiple sources, enrich your data and collect it in your data-warehouse.
It can be installed on your cloud provider or on-premise servers and we can manage your Rakam API cluster for you. 
Your data is stored as part of the Rakam API, you send the data and execute your queries on it

Rakam UI is our user interface solution for Rakam API. It allows you to run your queries with a simple interface, visualize the data,
create custom reports and dashboards. Similar to other analytics services, you can visualize segmentation, funnel and retention reports, 
search users or see the timeline of a user. You may think Rakam UI similar other BI solutions such as Tableau since it allows you to
visualize the data and create charts but that's not the point: 
It's a tool for creating user interface for your custom analytics service. 
The interface of Rakam UI is similar to other event analytics services such as Mixpanel and Amplitude. 
It doesn't know anything about your data, it just knows that there are events with at least two attributes; _user and _time. 
If you're familiar with Google Analytics, you may already know the metrics you have in web analytics. 
You can just create an analytics service similar to Google Analytics with Rakam UI if that's what you need. 
The [recipe mechanism](/doc/buremba/rakam-wiki/master/Rakam-BI/Recipes) is built for pre-defined analytics solutions, 
if you're a mobile gaming company and we have the recipe for your industry, 
we can give it to you so that you can import it and use Rakam as gaming analytics solution.

You can also use Rakam API without Rakam UI, similar to Keen.io. It exposes [an API](//api.rakam.io) 
for you so you can develop a custom interface using the API for your users, it's up to you. 
You can even plug your BI solutions with Rakam API and use it as a data warehouse.

