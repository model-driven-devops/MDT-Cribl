# Table of Contents

* [Dashboard Setup](dashboard-setup)
  * [Create Dashboard](#create-dashboard)
  * [Region Map](#region-map)

# Dashboard Setup

Now that data is coming into ElasticSearch, this section will go through the creation of some basic dashboards.

## Create Dashboard

Open up your Kibana instance and login. Once logged in, select the menu and navigate to Dashboards. Go ahead and select the "Create Dashboard" button. This is going to be where we add our visualizations.

![Elastic-01](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/5a98bb01-aa69-46e3-97ee-3b1c398f005c)

Once you select "Create Dashboard", select "Create Visulization" to get started. You'll see all the awesome and simplified field options in the left bar and on the right will be where we select which fields we want to visualize. 

![Screenshot 2024-01-25 at 9 13 30 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/ef501ebf-e514-427d-844b-06312bf694b5)

One helpful feature is the ability to click any field and see what data is available for visualizing.

![Screenshot 2024-01-25 at 9 15 00 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/bfb5c208-c00d-45cb-a931-a8c6f802594a)

### Metrics

Lets start by generating some basic metrics. Select the drop down menu on top of the empty visualization and select "Metric".

![Screenshot 2024-01-25 at 9 27 09 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/68aee205-17e2-45df-870f-be70dd21ce4b)

If you recall in earlier excercises, we cleaned up our data so "value" is its own field. What that means is any of the other fields can be used as almost a filter to display your "values".

![Screenshot 2024-01-25 at 9 36 07 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/4a174de7-8fe8-41a7-a52a-5c7a999820b9)

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

### Region Map

Since we took some extra steps to get our geo-location sent into Elastic, lets start by creating a cool map to overlay some data on top of. If you hover over the location field, you'll see our correctly populated geo-points (I'm pretty excited about this).

![Screenshot 2024-01-25 at 9 22 43 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/f2d52f9a-6975-42e6-ac63-08838a34dc01)
![Screenshot 2024-01-25 at 9 27 09 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/68aee205-17e2-45df-870f-be70dd21ce4b)
