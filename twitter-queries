-- Create twtr database
CREATE DATABASE IF NOT EXISTS twtr;

-- Create Tweets Table
-- This is an external table that points to AWS S3 staging location of all incoming tweets. Please change it as necessary.
DROP TABLE IF EXISTS twtr.tweets;
CREATE EXTERNAL TABLE IF NOT EXISTS twtr.tweets (
  author_id string,
  name string,
  username string,
  tweet_id string,
  created_at string,
  lang string,
  text string
)
ROW FORMAT SERDE
    'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    'escapeChar'='\\',
    'quoteChar'='"',
    'separatorChar'='|')
STORED AS INPUTFORMAT
    'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
    'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3a://pse-uat2/twtr/'
;

-- Twitter View
-- This view marries tweets data with ISO Language Codes, to help with languages pie chart in the Twitter Dashboard.
-- This also removes the duplicate tweets.
DROP VIEW IF EXISTS twtr.twtr_view;
CREATE VIEW IF NOT EXISTS twtr.twtr_view AS
SELECT DISTINCT
  a.author_id as author_id,
  a.name as name,
  a.username as username,
  a.tweet_id as tweet_id,
  FROM_UTC_TIMESTAMP(UNIX_TIMESTAMP(a.created_at, "yyyy-MM-dd'T'hh:mm:ss.SSS'Z'") * 1000, 'PST') as created_at,
  a.lang as lang,
  b.lang_name as lang_name,
  a.text as text
FROM twtr.tweets a
JOIN twtr.iso_language_codes b on b.lang_code = a.lang;

-- Tweets By Minute View
-- This view groups the number of tweets by minute, to build a timeline in the Twitter Dashboard.
DROP VIEW IF EXISTS twtr.tweets_by_minute;
CREATE VIEW IF NOT EXISTS twtr.tweets_by_minute AS
WITH tweets_custom as (
  SELECT
    *,
    from_unixtime(UNIX_TIMESTAMP(created_at, "yyyy-MM-dd HH:mm:ss"),"yyyy-MM-dd HH:mm") as date_hhmm
  FROM twtr.twtr_view
)
SELECT
  date_hhmm as formatted_date,
  count(*) as tot_tweets
FROM tweets_custom
GROUP BY date_hhmm
ORDER BY date_hhmm desc;
