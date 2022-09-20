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

### Step #1 - Cloudera DataFlow (CDF)
- Go to CDF user interface, and ensure CDF service is enabled in your CDP environment.
- Import the following flow definition - [NiFi_Twitter_Flow.json](/NiFi_Twitter_Flow.json)
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

### Step #2 - Cloudera Data Warehouse (CDW)
- Go to CDW user interface. Ensure CDW service is activated in your CDP environment, and a Database Catalog & a Virtual Warehouse compute cluster are available for use.
- In Hue editor, manually load [ISO Language Codes](/data/ISO%20Language%20Codes.csv) into a table. Default settings in the importer wizard will work fine. If you're not sure how to upload data in Hue, visit [Hue Importer -- Select a file, choose a dialect, create a table](https://gethue.com/blog/2021-05-26-improved-hue-importer-select-a-file-choose-a-dialect-create-a-table/).
- In Hue editor, execute [twitter-queries.sql](/twitter-queries.sql). This will create the necessary tables and views, required to support the visuals in the Twitter Dashboard. **Please change AWS S3 location where you've staged the tweets data.**
- After the query execution is successful, you will be able to validate tables using queries below.
  ```sql
  SELECT * FROM twtr.iso_language_codes a;
  SELECT * FROM twtr.tweets b;
  SELECT * FROM twtr.twtr_view c;
  SELECT * FROM twtr.tweets_by_minute d;
  ```

### Step #3 - Data Visualization (Dataviz)

