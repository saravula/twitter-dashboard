# Twitter Dashboard
## Use Case
Real-time data visualization to analyze Twitter feeds.

## Design
![Design - Twitter Dashboard](/assets/design-Twitter-dashboard.png)

**Explanation:**
- NiFi Flow invokes Twitter API v2 (every 15 seconds), and stages all tweets in an AWS S3 bucket.
- Hive External Tables points to the staging location (i.e. AWS S3 bucket).
- Data Visualization uses Hive External Tables as its data source, to create visuals. All visuals are refreshed every 20 seconds.

## Implementation
### Prerequisites
- A CDP Public Cloud environment on Amazon Web Services (AWS). If you don't have an existing environment, follow instructions here to set one up - [CDP/AWS Quick Start Guide](https://docs.cloudera.com/cdp-public-cloud/cloud/aws-quickstart/topics/mc-aws-quickstart.html).
- An app in [Twitter's Developer Portal](https://developer.twitter.com/en/portal/dashboard). This is needed to call Twitter API v2. If this is your first time using Twitter API v2, follow these instructions - [Step-by-step guide to making your first request to the new Twitter API v2](https://developer.twitter.com/en/docs/tutorials/step-by-step-guide-to-making-your-first-request-to-the-twitter-api-v2).
---
### Step #1 - Cloudera DataFlow (CDF)
- Go to CDF user interface, and ensure CDF service is enabled in your CDP environment.
- Import the following flow definition - [nifi-twitter-flow.json](/nifi-twitter-flow.json)
- Select imported flow, click on Deploy, select the Target Environment and begin the deployment process.
- During the deployment, it's going to ask about the following parameters that this NiFi Flow requires to function:
  - **AWS - Access Key ID** - visit [Understanding and getting your AWS credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) if you're not clear on how to get it. Ensure that AWS IAM user you're using, has "AmazonS3FullAccess" permissions.
  - **AWS - Secret Access Key** - same instructions as **AWS - Access Key ID**.
  - **AWS S3 Bucket** - provide AWS S3 bucket name. Ensure that IAM user has access to this S3 bucket.
  - **AWS S3 Bucket Subdirectory** - provide subdirectory in AWS S3 bucket where you want to stage your tweets. 
    > It's usually best to delete any historical data from this subdirectory, so you're only staging latest tweets.
  - **Twitter API v2 Bearer Token** - provide your app's bearer token from Twitter's Developer Portal.
  - **Twitter Search Term** - provide the search term for which you want to do the analysis. For ex: COVID19, DellTechWorld, IntelON, etc. Only one search term is allowed at the moment.
- Extra Small NiFi node size is enough for this data ingestion.
- After deployment is done, you would be able to see the flow in Dashboard.
- Open NiFi Flow to understand how it's working.

  ![Screen Shot 2022-09-20 at 3 20 59 PM](https://user-images.githubusercontent.com/2523891/191375477-84262a11-622f-4026-bfac-ac908c2d8931.png)
- Notes are available in NiFi Flow to help you understand the use of each processor.

  ![Screen Shot 2022-09-20 at 3 25 39 PM](https://user-images.githubusercontent.com/2523891/191375811-dd24c63e-911e-4bf0-bc67-1b531021fb7f.png)
- All NiFi Flow parameters can be updated while the flow is running, from Deployment Manager. As soon as you Apply Changes, running processors that are affected by the Parameter changes will automatically be restarted.

  ![Screen Shot 2022-09-20 at 3 36 28 PM](https://user-images.githubusercontent.com/2523891/191377135-4317c855-4afd-4704-bd1e-45e7bdc811f9.png)
---
### Step #2 - Cloudera Data Warehouse (CDW)
- Go to CDW user interface. Ensure CDW service is activated in your CDP environment, and a Database Catalog & a Virtual Warehouse compute cluster are available for use.
- In Hue editor, manually load [ISO Language Codes](/data/ISO%20Language%20Codes.csv) into a table. Default settings in the importer wizard will work fine. If you're not sure how to upload data in Hue, visit [Hue Importer -- Select a file, choose a dialect, create a table](https://gethue.com/blog/2021-05-26-improved-hue-importer-select-a-file-choose-a-dialect-create-a-table/).
- In Hue editor, execute [twitter-queries.sql](/twitter-queries.sql). This will create the necessary tables and views, required to support the visuals in the Twitter Dashboard. **Please change AWS S3 location to where you've staged the tweets data.**
- After the query execution is successful, you will be able to validate tables using queries below.

  ```sql
  SELECT * FROM twtr.iso_language_codes a;
  SELECT * FROM twtr.tweets b;
  SELECT * FROM twtr.twtr_view c;
  SELECT * FROM twtr.tweets_by_minute d;
  ```
---
### Step #3 - Data Visualization
- Go to CDW user interface, select Data Visualization and add a new Data VIZ.
- In Data Visualization user interface, create a new connection. You must be logged in as admin to create a new connection.

  ![Screen Shot 2022-09-20 at 5 00 14 PM](https://user-images.githubusercontent.com/2523891/191385311-01144e7c-63c4-4a4d-9334-204411f048d4.png)
- Now that you have a connection to Hive virtual warehouse, let's create two datasets required to support the visuals.
- **Create first dataset:**
  - Dataset Title - Twitter View
  - Dataset Source - From Table
  - Select Database - twtr
  - Select Table - twtr_view

  ![Screen Shot 2022-09-20 at 5 13 32 PM](https://user-images.githubusercontent.com/2523891/191386839-de3ae0e1-8da5-487e-bb96-8811c1b7e1eb.png)
- **Create second dataset:**
  - Dataset Title - Tweets By Minute
  - Dataset Source - From Table
  - Select Database - twtr
  - Select Table - tweets_by_minute

  ![Screen Shot 2022-09-20 at 5 18 28 PM](https://user-images.githubusercontent.com/2523891/191387159-9fae6ddb-17f1-409c-922b-b23b7a9479ba.png)
- It's now time to Import Visual Artifacts. Take a quick look at [Importing a dashboard](https://docs.cloudera.com/data-visualization/7/howto-dashboards/topics/viz-import-dashboard.html) if you're doing it for the first time. 

  ![Screen Shot 2022-09-20 at 5 03 37 PM](https://user-images.githubusercontent.com/2523891/191385727-13514315-05e8-493c-adf6-37e6ef3521c5.png)
- Choose [dataviz-twitter-dashboard.json](/dataviz-twitter-dashboard.json) in the import dialog.

  ![Screen Shot 2022-09-20 at 5 45 41 PM](https://user-images.githubusercontent.com/2523891/191389616-45bda939-8a41-489f-8547-0d1ad8101bdd.png)
- Once you get the following screen, click ACCEPT AND IMPORT.

  ![Screen Shot 2022-09-20 at 5 49 11 PM](https://user-images.githubusercontent.com/2523891/191389977-158e90ad-32b1-41e1-b31a-058648e1ebe1.png)
- Twitter Dashboard should be successfully imported at this point. To see it, go to VISUALS from the top menu and select Twitter Dashboard.
- **Congratulations on creating your real-time Twitter Dashboard using Cloudera Data Platform!!!** To learn more about its implementation, please register [here](https://attend.cloudera.com/skillupseriesoctober20) to watch the recording.
  
  ![Twtr Dashboard](https://user-images.githubusercontent.com/2523891/191391831-2347602b-02b3-46dc-889f-ea178d3a1b27.png)
---
