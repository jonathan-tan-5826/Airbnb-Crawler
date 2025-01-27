# Airbnb Crawler

A multi-threaded crawler written in Java to obtain the average price per month per zipcode for renting homes through Airbnb.

Technologies used:
Selenium-Java 2.53
Maven 3.0.2
MySQL-Connector-Java 5.1.6

Remember to setup xvfb:

display=`id -u`
Xvfb :${display} -screen 0 1920x1080x24 &
export DISPLAY=:${display}

Screen must be at least 1920x1080.

General explanation:
1) Pass in a state abbreviation or group number (1-10), month, year, desired number of threads, and 1 to test IP or 0 not to (Ex: java -jar airbnb_crawler_v2.jar CA 10 2017 10 0)
2) Queries the database for all zipcodes in that state, along with their corresponding cities, and distributes the CityZip amongst X buckets (X = passed in desired number of threads).
3) For each bucket with CityZips, a worker thread is assigned to crawl Airbnb for each zipcode in the bucket for the month and year passed in.

Inside the worker threads:
For each zipcode in the worker's pass in bucket, the worker performs the following...
1) Loads URL in Firefox driver
2) Verifies the city listed on the webpage matches the zipcode's city. If not, the worker continues to the next zipcode.
3) Verifies there are at least 5 listings on the page. If not, the worker continues to the next zipcode.
4) Verifies the period (Makes sure the average price is for the month). If not, the worker continues to the next zipcode.
5) Waits for price range button to appear using new selector. If the button does not appear, attempts to get price range button with the old selector. If both fail to appear, increment crawlAttempts (initially 0). If crawlAttempts < TOTAL_RETRIES, worker restarts at step 1 for this zipcode. If crawlAttempts >= TOTAL_RETRIES, worker continues to the next zipcode.
6) Worker clicks the price button and waits for price-text to appear to obtain the average price.
6a) If the price button was obtained using the old selector, validates the city again. If the city is invalid, the worker continues to the next zipcode.
7) Worker saves page source.
8) Worker obtains the average price from the price-text and saves the information to database.

Testing the crawler:
Use the following command to test the crawler.
java -jar airbnb_crawler_v2.jar test 1 2017 1 1
