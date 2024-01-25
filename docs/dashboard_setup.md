# Table of Contents

* [Dashboard Setup](dashboard-setup)
  * [Create Dashboard](#create-dashboard)
  * [Create Metrics](#create-metrics)
  * [Create Graphs](#create-graphs)
  * [Create Map](#create-map)

# Dashboard Setup

Now that data is coming into ElasticSearch, this section will go through the creation of some basic dashboards.

## Create Dashboard

Open up your Kibana instance and login. Once logged in, select the menu and navigate to Dashboards. Go ahead and select the "Create Dashboard" button. This is going to be where we add our visualizations.
<p align="center">
<img src='https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/5a98bb01-aa69-46e3-97ee-3b1c398f005c' width=70% height=70%>
</p>

Once you select "Create Dashboard", select "Create Visulization" to get started. You'll see all the awesome and simplified field options in the left bar and on the right will be where we select which fields we want to visualize. 
<p align="center">
<img src='https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/ef501ebf-e514-427d-844b-06312bf694b5' width=70% height=70%>
</p>
One helpful feature is the ability to click any field and see what data is available for visualizing.
<p align="center">
<img src='https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/bfb5c208-c00d-45cb-a931-a8c6f802594a' width=70% height=70%>
</p>

## Create Metrics

Lets start by generating some basic metrics. Select the drop down menu on top of the empty visualization and select "Metric".

<p align="center">
<img src='https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/68aee205-17e2-45df-870f-be70dd21ce4b' width=70% height=70%>
</p>

If you recall in earlier excercises, we cleaned up our data so "value" is its own field. What that means is any of the other fields can be used as almost a filter to display your "values".

<p align="center"
<img src='https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/4a174de7-8fe8-41a7-a52a-5c7a999820b9' width=70% height=70%>
</p>

In this example, we've selected "bgp.name.keyword" which shows us the fields tied to our BGP telemetry stream, along with the "value" associated with each field. 

![Screenshot 2024-01-25 at 9 39 32 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/8c23249c-f0dd-4b74-8fa0-defe52bc023b)

To keep it simple, and because we are using this to visualize streaming telemetry (live data), we are going to select the "Last Value of Data" and then we are going to break the data down by the top 5 values.

![Screenshot 2024-01-25 at 9 47 01 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/3aada136-a7f2-415a-a67d-95cab2026321)

Now lets try a field that contains more data. Drag "interface.name.keyword" onto the screen. Your metric boxes will reset. On the right side, move interface.name.keyword from "Primary Metric" to "Break down by". Now lets grab the "value" field from the left column and drag it to "Primary Metric". You'll see your interface data populate. 

![Screenshot 2024-01-25 at 9 56 34 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/c17deaa4-b487-4756-a8d8-e8b0c06818d7)

Now lets clean it up. Select the "value" field under "primary metric". Change the function from "median" to "Last Value" to monitor the live data.

![Screenshot 2024-01-25 at 9 59 36 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/d3175911-07fe-4ea3-8fd4-bfccdf39f998)

Now scroll down to "Appearance" and change the format to "Bytes". Since we are feeling nerdy, lets add a neat compute icon :).

![Screenshot 2024-01-25 at 10 00 56 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/8cc78953-bd03-4be8-ba54-c8dc7ae7ab0c)

Well will you look at that. Now we can see the bandwidth going in and out of our routers. It's almost like Elastic is made for this!

![Screenshot 2024-01-25 at 10 02 22 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/c088db42-3d25-47f0-8c4b-c1145afd99c8)

Go ahead and save, which will add it to your dashboard. Give it a name and lets move on to the next visualization!

## Create Graphs

What's better than seeing live metrics? That's right! Metrics over time. Everyone loves seeing those peaks and valleys. Lets add another visualization but this time select the "Line" chart from the top. Now we want to start with using the timestamp as our horizontal access. 

![Screenshot 2024-01-25 at 10 12 04 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/92ab2c97-2b0f-45b7-bded-5d4c552e1e03)

Now lets drag our "value" field to the vertical access. Once you do that, select it and choose the "Last value" of value.

![Screenshot 2024-01-25 at 10 15 31 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/55f59434-6452-443d-af9c-e9aa0cf99ab0)

Now this is the cool part having organized data. You can really use this exact same line chart and replace it with any telemetry stream you want. Let's do SLAs for this first one. Drag the "sla.name.keyword" to breakdown. 

![Screenshot 2024-01-25 at 10 23 45 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/ac85cb2d-7384-4e79-b862-43c2ad00b6a0)

You may notice the "other" line on the chart. If you want to remove any of these fields, all you have to do is select it and filter it out.

![Screenshot 2024-01-25 at 10 25 06 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/8b493a21-fad4-4c88-9308-58eafc9ffa3a)

Go ahead and save the SLA line chart to your dashboard. Once saved, select it in the dashboard and clone it. On the cloned panel, click the gear in the top right and select "Edit Lens".

![Screenshot 2024-01-25 at 10 26 55 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/c05518e2-f06c-4cd1-9612-d386b72eeb1f)

Remove the filter from the top, which was specfically for SLAs. Drag your interface.name.keyword field to the breakdown box. You'll see the interface lines pop up. Now go to Last value of value and change the metric to Bytes again.

![Screenshot 2024-01-25 at 10 32 53 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/41f16d8c-4936-4e8c-b489-6c97ec8298a9)

Not much traffic flowing through our simulated network anymore. Go ahead and filter out any fields you don't want. Then, depending on how long your telemetry has been collecting, go to the top and change the value of your time. 

![Screenshot 2024-01-25 at 10 35 05 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/ee0eaa09-3bde-4ae1-8325-d57ed7c4d241)

In the above example, I am now going back 15 days. You can see where my simulation was probably shut down and then turned back on. Go ahead and add any additional filters and then save it to the dashboard.

## Create Map

Since we took some extra steps to get our geo-location sent into Elastic, lets start by creating a cool map to overlay some data on top of. If you hover over the location field, you'll see our correctly populated geo-points (I'm pretty excited about this). Use the sidebar to nagivate to "Maps".

![Screenshot 2024-01-25 at 12 41 08 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/88ef16ff-3b37-4930-8ff5-dae0a011701c)

Select "Add Layer" and then selet "Documents". Now we want to select our dataset and then our Geolocation field.

![Screenshot 2024-01-25 at 12 42 58 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/5a3f988e-e6a4-4827-93ae-c61a29486d3a)

Create a new layer. Add some tooltips that will be pop ups on the map that show the data you're looking for.

![Screenshot 2024-01-25 at 12 58 20 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/6989fa7a-0943-4a4d-b362-24979a11c71d)

Go ahead and addd it to your dashboard. We will come back to this in another excercise. For now we just want to get our dashboard looking good.

![Screenshot 2024-01-25 at 1 00 39 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/17bc0151-e3c3-4124-a27f-c4140372f19d)

Now you just have to drag and organize all the visualizations the way you want them layed out. Once you do that, you can start adding some filters at the top using the "Controls" option.

![Screenshot 2024-01-25 at 1 02 31 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/cffadf95-6d13-40b6-8903-fd4335dd27ad)

There you go. Now you have a working dashboard displaying your telemetry.

