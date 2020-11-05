# Designing a URL Shortening service like TinyURL
> Let's design a URL shortening service like TinyURL. This service will provide short aliases redirecting to long URLs. Similar services: bit.ly, goo.l, qlink.me, etc. Difficulty Level: Easy

## 1. Why do we need URL shortening?
URL shortening is used to create shorter aliases for long URLs. We call these shortened aliases “short
links.” Users are redirected to the original URL when they hit these short links. Short links save a lot of
space when displayed, printed, messaged, or tweeted. Additionally, users are less likely to mistype
shorter URLs.

## 2. Requirements and Goals of the System
__Functional Requirements:__
- Given a URL, our service should generate a shorter and unique alias of it. This is called a short
link.
- When users access a short link, our service should redirect them to the original link.
- Users should optionally be able to pick a custom short link for their URL.
- Links will expire after a standard default timespan. Users should be able to specify the
expiration time.
__Non-Functional Requirements:__
- The system should be highly available. This is required because, if our service is down, all the
URL redirections will start failing.
- URL redirection should happen in real-time with minimal latency.
- Shortened links should not be guessable (not predictable).
__Extended Requirements:__
- Analytics; e.g., how many times a redirection happened?
- Our service should also be accessible through REST APIs by other services. 

## 3. Capacity Estimation and Constraints
Our system will be read-heavy. There will be lots of redirection requests compared to new URL
shortenings. Let’s assume 100:1 ratio between read and write.
- Traffic estimates
- Storage estimates
- Bandwidth estimates
- Memory estimates

__High level estimates__
Assuming 500 million new URLs per month and 100:1 read:write ratio,
following is the summary of the high level estimates for our service:
`New URLs 200/s |
URL redirections 20K/s |
Incoming data 100KB/s |
Outgoing data 10MB/s |
Storage for 5 years 15TB |
Memory for cache 170GB`

## 4. System APIs
> createURL(api_dev_key, original_url, custom_alias=None, user_name=None,
expire_date=None)

__Parameters:__
api_dev_key (string): The API developer key of a registered account. This will be used to, among other
things, throttle users based on their allocated quota.
original_url (string): Original URL to be shortened.
custom_alias (string): Optional custom key for the URL.
user_name (string): Optional user name to be used in encoding.
expire_date (string): Optional expiration date for the shortened URL.
__Returns: (string)__
A successful insertion returns the shortened URL; otherwise, it returns an error code.

> deleteURL(api_dev_key, url_key)
Where “url_key” is a string representing the shortened URL to be retrieved. A successful deletion
returns ‘URL Removed’.

## 5. Database Design
A few observations about the nature of the data we will store:
- We need to store billions of records.
- Each object we store is small (less than 1K).
- There are no relationships between records—other than storing which user created a URL.
- Our service is read-heavy. 
__Database Schema:__
We would need two tables: one for storing information about the URL mappings, and one for the user’s
data who created the short link.
__What kind of database should we use?__ Since we anticipate storing billions of rows, and we don’t
need to use relationships between objects – a NoSQL key-value store like DynamoDB, Cassandra or
Riak is a better choice. A NoSQL choice would also be easier to scale. Please see SQL vs NoSQL for
more details.

## 6. Basic System Design and Algorithm
The problem we are solving here is, how to generate a short and unique key for a given URL.

In the TinyURL example in Section 1, the shortened URL is “http://tinyurl.com/jlg8zpc”. The last six
characters of this URL is the short key we want to generate. We’ll explore two solutions here:
a. Encoding actual URL
b. Generating keys offline

## 7. Data Partitioning and Replication
a. Range Based Partitioning
b. Hash-Based Partitioning

## 8. Cache

## 9. Load Balancer (LB)
We can add a Load balancing layer at three places in our system:
- Between Clients and Application servers
- Between Application Servers and database servers
- Between Application Servers and Cache servers 

## 10. Purging or DB cleanup
Whenever a user tries to access an expired link, we can delete the link and return an error to the
user.
- A separate Cleanup service can run periodically to remove expired links from our storage and
cache. This service should be very lightweight and can be scheduled to run only when the user
traffic is expected to be low.
- We can have a default expiration time for each link (e.g., two years).
- After removing an expired link, we can put the key back in the key-DB to be reused.
- Should we remove links that haven’t been visited in some length of time, say six months? This
could be tricky. Since storage is getting cheap, we can decide to keep links forever. 

## 11. Telemetry
Some statistics worth tracking: country of the visitor, date and time of access, web page that refers the
click, browser, or platform from where the page was accessed.