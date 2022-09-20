# Twitter Dashboard
## Use Case
Real-time data visualization to analyze Twitter feeds.

## Design
![Design - Twitter Dashboard](/assets/design-Twitter-dashboard.png)

**Explanation:**
- NiFi Flow invokes Twitter API v2 (every 15 seconds), and stages all tweets in an AWS S3 bucket.
- Hive External Tables points to the staging location (i.e. AWS S3 bucket).
- Data Visualization uses Hive External Tables as its data source, to create visuals and refresh them every 20 seconds.

## Implementation
### Prerequisites
- A CDP Public Cloud environment on Amazon Web Services (AWS). If you don't have an existing environment, follow instructions here to set one up - [CDP/AWS Quick Start Guide](https://docs.cloudera.com/cdp-public-cloud/cloud/aws-quickstart/topics/mc-aws-quickstart.html).
- An app in [Twitter's Developer Portal](https://developer.twitter.com/en/portal/dashboard). This is needed to call Twitter API v2. If this is your first time using Twitter API v2, follow these instructions - [Step-by-step guide to making your first request to the new Twitter API v2](https://developer.twitter.com/en/docs/tutorials/step-by-step-guide-to-making-your-first-request-to-the-twitter-api-v2).

### Step #1 - Cloudera DataFlow

### Step #2 - Cloudera Data Warehouse

### Step #3 - Data Visualization

