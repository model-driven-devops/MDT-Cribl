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



### Region Map

Since we took some extra steps to get our geo-location sent into Elastic, lets start by creating a cool map to overlay some data on top of. If you hover over the location field, you'll see our correctly populated geo-points (I'm pretty excited about this).

![Screenshot 2024-01-25 at 9 22 43 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/f2d52f9a-6975-42e6-ac63-08838a34dc01)
![Screenshot 2024-01-25 at 9 27 09 AM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/68aee205-17e2-45df-870f-be70dd21ce4b)
