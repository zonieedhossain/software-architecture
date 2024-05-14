# Basic Software Design Document for Generic Web Crawler System (Public Version)

## Team

- **Contacts:** [zonieed.uap@gmail.com](mailto:zonieed.uap@gmail.com)
- **Location:** Dhaka, Bangladesh


## Introduction

The generic web crawler system standardizes data collection, reduces maintenance complexity, and simplifies crawling across various websites. Its goals include:

- **Up-to-date Data Collection:** Fetch data from different websites to get the latest information.
- **Extensible:** Add support for additional media types with future upgrades.
- **Flexible & Scalable:** Adapt to both current and new websites with ease.

## System Requirements

- **Data Collection:** Efficiently collect data from various websites, handling diverse content formats.
- **Scalability:** Accommodate increasing data volumes and website additions.
- **Maintainability:** Modular and well-documented codebase for ease of maintenance and future development.
- **Security:** Implement robust security measures to protect collected data.

## Recommended Platform

### Cloud: Google Cloud Platform (GCP)

### Core Services

- **BigQuery:** Efficiently analyze large datasets to uncover trends.
  - [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- **Cloud Storage:** Store raw and processed crawl data at scale.
  - [Cloud Storage Documentation](https://cloud.google.com/storage/docs)
- **Compute Engine:** Host and deploy custom crawling applications.
  - [Compute Engine Documentation](https://cloud.google.com/compute/docs)
- **Kubernetes Engine:** Orchestrate and scale containerized crawling applications.
  - [Kubernetes Engine Documentation](https://cloud.google.com/kubernetes-engine/docs)
- **Cloud Functions:** Trigger event-based tasks like rule updates or alerts.
  - [Cloud Functions Documentation](https://cloud.google.com/functions/docs)
- **Pub/Sub:** Facilitate event-driven communication between services.
  - [Pub/Sub Documentation](https://cloud.google.com/pubsub/docs)
- **Cloud Armor:** Secure and protect against threats.
  - [Cloud Armor Documentation](https://cloud.google.com/armor/docs)
- **Event Message Bus (RabbitMQ/NATS):** Manage real-time message processing.
  - [RabbitMQ Documentation](https://www.rabbitmq.com/documentation.html)
  - [NATS Documentation](https://docs.nats.io)

### Data Storage: Hybrid Approach

- **SQL Database (e.g., MySQL):** Manage structured data like URLs, crawl status, and page analytics.
  - [Google Cloud SQL (MySQL) Documentation](https://cloud.google.com/sql/docs/mysql)
- **NoSQL Database (e.g., MongoDB):** Store web content, images, or JSON documents for rapid scaling.
  - [MongoDB Atlas on Google Cloud](https://www.mongodb.com/atlas/google-cloud)

## Basic FlowChart for Web Crawler System
![Flowchart](https://github.com/zonieedhossain/software-architecture/blob/main/graphical-flowchart-crawler-design.png)

## Why We Are Using This Solution

- **Efficient Data Collection:** Standardizing the collection process ensures consistent, reliable data acquisition across multiple sources.
- **Cost Reduction:** Managing separate crawlers for individual websites is costly and difficult to maintain.
- **Scalability:** The system is designed to be flexible and scalable for current and future needs.
- **Security:** Implementing cloud-based security measures helps safeguard collected data.

# Implementation Details Using Golang
The system leverages Google Cloud Platform (GCP) for a distributed and highly available architecture. Here's a breakdown of the data flow and component responsibilities:

### Development
- **Language & Tools:** 
  - **Golang:** Leverage Golang's concurrency features and efficiency for building a scalable web crawler. (https://go.dev/).
  - **Libraries:** Utilize libraries like:
    - **Colly** for scraping functionalities (https://github.com/AlephAlpha/golly).
    - **goquery** for HTML parsing (https://github.com/PuerkitoBio/goquery).
    - **gorilla/mux** (optional) for routing if exposing RESTful APIs (https://github.com/gorilla/mux).
    
- **Features:**
  - **Politeness:** 
    - Respect robots.txt protocols by using libraries like go-robots-txt (https://github.com/temoto/robotstxt) or implementing custom parsing logic.
  - **Error Handling:** 
    - Robust error handling to manage timeouts, HTTP errors, and network issues, using Go's built-in error interface and custom error types for granular control.
  - **Middleware:** 
    - **Authentication:** Utilize libraries like go-oauth2 (https://github.com/golang/oauth2) or oauth1 (https://github.com/topics/oauth1) for authentication with different providers.
    - **Data Extraction:** Design custom middleware to extract specific data from web pages based on your scraping requirements.
    - **Rate Limiting:** Implement rate limiting using libraries like go-rate (https://github.com/golang/time/blob/master/rate/rate.go) to regulate website requests and avoid overloading servers.

### Example Code Snippet:
Here is a basic example of how you might set up a web crawler using Golang with the Colly library:
```go
package main

import (
  "errors"
  "fmt"
  "log"

  "github.com/PuerkitoBio/go-query" // Import goquery
  "github.com/gocolly/colly"
  "github.com/robinton/go-robots-txt" // Import go-robots-txt
)

func main() {
  c := colly.NewCollector(
    colly.AllowedDomains("example.com"),
  )

  // Respect robots.txt
  robotsTxt, err := robots.Parse("https://example.com/robots.txt")
  if err != nil {
    log.Fatal(err)
  }

  c.OnRequest(func(r *colly.Request) {
    if !robotsTxt.CanFetch(r.URL.String(), "*") {
      r.Abort() // Respect robots.txt disallow rules
      return
    }
    log.Println("Visiting", r.URL.String())
  })

  c.OnError(func(err error) {
    // Handle errors (timeouts, HTTP errors, etc.)
    fmt.Println("Error:", err)
  })

  c.OnHTML("a[href]", func(e *colly.HTMLElement) {
    link := e.Attr("href")
    log.Println("Link found:", link)
    e.Request.Visit(link)
  })

  c.OnHTML("h1", func(e *colly.HTMLElement) {
    title := e.Text
    log.Println("Title:", title)
  }) // Example data extraction (modify selectors as needed)

  c.Visit("https://example.com")
}

```
## Cloud Functions (Serverless)
- **Functionality:** 
  - Deploy serverless functions using GCP's Cloud Functions written in Go, which will respond to events from Pub/Sub to process data in real-time.
- **Tasks Include:**
  - **Data Transformation:** Utilize Golang's standard library and libraries like encoding/json or xml (https://pkg.go.dev/encoding/json) to parse and transform raw data into structured formats suitable for storage and analysis.
  - **Data Normalization:** Standardize and validate data formats (e.g., date formats, units) to ensure consistency within your data pipeline. You can define data validation rules within Cloud Functions.
  - **Event Triggers:** Golang's concurrency model facilitates efficient handling of event triggers from Pub/Sub, enabling high-throughput message processing. Refer to the Cloud Functions documentation for event triggers (https://cloud.google.com/functions/docs/calling).
### Example Code Snippet:
```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"

    "cloud.google.com/go/functions/v1"
    "cloud.google.com/go/sql/driver"
)

type CrawledURL struct {
    URL string `json:"url"`
}

func main() {
    ctx := context.Background()

    // (Connect to Cloud SQL database)
    db, err := driver.Open("mysql", "user:password@tcp(your-project-id.cloudsql.com:3306)/your-database")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // (Function triggered by Pub/Sub message)
    f := func(ctx context.Context, message *functions.CloudEvent) error {
        var url CrawledURL
        err := json.Unmarshal(message.Data, &url)
        if err != nil {
            log.Println("Error unmarshalling message:", err)
            return nil // Handle errors appropriately
        }

        // Insert URL into database
        stmt, err := db.PrepareContext(ctx, "INSERT INTO urls (url) VALUES (?)")
        if err != nil {
            log.Println("Error preparing statement:", err)
            return nil // Handle errors appropriately
        }
        _, err = stmt.ExecContext(ctx, url.URL)
        if err != nil {
            log.Println("Error inserting URL:", err)
            return nil // Handle errors appropriately
        }
        fmt.Println("Inserted URL:", url.URL)
        return nil
    }

    // Register Cloud Function
    log.Fatal(functions.Register(ctx, f))
}
```
## Data Storage (Hybrid Approach)
### Structured Data Storage Cloud SQL (MySQL): 
  - **Functionality:** Ideal for storing well-defined, schema-based data commonly extracted during web crawling. This includes:
    - URLs (seed URLs, crawled URLs, filtered URLs).
    - Crawl statuses (success, failure, retry count).
    - Analytics data (extracted product details, website statistics).
- **Benefits:** 
  - Efficient querying and retrieval using SQL.
  - Relational capabilities for linking related data entries.
  - Scalability to handle large volumes of structured data.
- **Golang Integration:**
  - Utilize the database/sql package in Golang to connect, interact with, and manage data within your Cloud SQL instance. Refer to the official documentation for detailed instructions: https://pkg.go.dev/database/sql.
- **Documentation:**
  - Golang database/sql package: https://pkg.go.dev/database/sql.

### Unstructured Data Storage MongoDB Atlas on GCP:
- **Functionality:** Suitable for storing variable-length, non-rigid data structures like:
  - Web page content (HTML snippets, text extracts).
  - Images (product images, logos).
  - JSON or XML data (if encountered during crawling).
- **Benefits:**
  - Flexible schema for accommodating diverse data types.
  - Scalability to handle large volumes of unstructured data.
- **Golang Integration:**
  - Use the mongo-go-driver package to interact with your MongoDB Atlas instance in Golang. Refer to the driver's documentation for installation and usage guides: https://github.com/mongodb/mongo-go-driver.
- **Documentation:**
  - mongo-go-driver: https://github.com/mongodb/mongo-go-driver.
  
## Data Orchestration (Pub/Sub)
- **Functionality:** Implement Google Cloud Pub/Sub (Pub/Sub) as a message queuing system to manage the flow of data between different components of your web crawler architecture.
- **Benefits:**
  - **Decoupling:** Pub/Sub decouples components, allowing them to operate independently and scale based on their needs.
  - **Scalability:** Pub/Sub can handle high volumes of messages, ensuring your system can handle increasing crawl demands.
  - **Fault Tolerance:** Messages are persisted in Pub/Sub topics, ensuring delivery even if a subscriber is temporarily unavailable.
- **Components:**
  - **Topics:** Create topics in Pub/Sub to represent different categories of messages (e.g., "crawled_urls", "extracted_data").
  - **Subscriptions:** Create subscriptions that bind your Cloud Functions and other services to specific topics. Subscribers receive messages published to their corresponding topics.
- **Golang Client Library:** 
  - Publish: Publish messages (e.g., URLs to be crawled, extracted data) from your Golang web crawler to relevant Pub/Sub topics.
  - Subscribe: Implement Cloud Functions or other services as subscribers that listen for messages on specific topics.
  - Process: Cloud Functions will receive messages triggered by Pub/Sub and perform necessary data processing tasks (e.g., transformation, normalization) before storing data.

### Example Code Snippet:
```go
package main

import (
    "context"
    "fmt"
    "log"

    "cloud.google.com/go/pubsub/v1"
)

func main() {
    ctx := context.Background()
    projectID := "your-project-id"

    // Create a Pub/Sub client
    client, err := pubsub.NewClient(ctx, projectID)
    if err != nil {
        log.Fatal(err)
    }

    // Define a topic and subscription
    topic := client.Topic("crawled_urls")
    subscription := client.Subscription("url_processor")

    // Publish a message to the topic
    data := []byte("https://example.com")
    result := topic.Publish(ctx, data)
    msgID, err := result.Get(ctx)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Published message: %d\n", msgID)

    // (In a separate Cloud Function)
    // Subscribe to the topic and process messages
    // ...
}
```
## Data Analysis and Retrieval (BigQuery)
- **Analytics:** Use BigQuery for performing heavy-duty analytics on the stored data. Leverage Golang's BigQuery client library to run queries and retrieve results for reporting or further analysis.

## Load Balancing
- **HTTP(S) Load Balancing:** Implement load balancing within GCP to distribute incoming requests across multiple instances of the web crawler, ensuring optimal resource utilization and redundancy.
- **High Availability:** Ensure high availability by redirecting requests to healthy containers if one becomes unavailable, minimizing downtime.
- **Efficient Resource Utilization:** Achieve an even distribution of requests across containers to prevent overload and maximize resource utilization.

# Metrics

## Success Metrics

The following metrics will be used to measure the success of the Generic Web Crawler System:

### Reduced Operational Costs
- **Cost Reduction:** Track the cost reduction achieved through this system compared to managing individual crawlers for each website. This could involve monitoring server resource utilization, personnel hours saved on maintenance, and any potential licensing cost savings.
```python
Cost_Reduction (%) = ((Cost_Before - Cost_After) / Cost_Before) * 100
```
### Increased Data Collection Efficiency
- **Data Volume:** Monitor the amount of data collected over time to evaluate if the system is gathering more data compared to previous methods.
```python
Crawl_Time = (End_Time - Start_Time) / Number_of_URLs_Crawled
```
- **Crawl Time:** Measure the average time taken to crawl a specific set of websites. A decrease in crawl time indicates improved efficiency.
```python
Average_Crawl_Time (seconds) = Total_Time_to_Crawl / Total_Pages_Crawled
```
- **Data Freshness:** Track how frequently new data is collected to ensure the system stays up-to-date.

### Reduced Maintenance Overhead
- **Maintenance Time:** Monitor the time and resources required for maintaining the Generic Web Crawler System. This includes patching software vulnerabilities, updating configurations, and addressing any operational issues. This should be significantly lower compared to managing multiple individual crawlers.

## Regression Metrics

To detect performance regressions, the system will monitor the following:

### Crawl Latency
- **Average Crawl Time:** Track the average time it takes for the crawlers to download and process a web page. An increase in latency could indicate potential performance bottlenecks.

### System Uptime
- **Uptime Percentage:** Monitor system uptime to ensure consistent data collection.
```python
Uptime_Percentage (%) = (Total_Operational_Time / Total_Time_Observed) * 100
```
### Error Rate
- **Crawl Error Frequency:** Track the frequency of crawl errors encountered. An increase in errors might suggest website changes or issues with the crawling logic.
```python
Crawl_Error_Frequency = Number_of_Errors / Number_of_Crawl_Attempts
```
### Data Completeness
- **Completeness Check:** Verify that all desired data elements are being extracted from web pages. Any missing data could indicate problems with the extraction process.

# Testing Plan

## Functional Testing:
- Verify that web crawlers can successfully download and extract data from various websites with different structures and content formats.
- Test data processing functionalities in Cloud Functions, ensuring accurate data transformation, normalization, and filtering.
- Validate data storage and retrieval processes in MySQL and MongoDB databases.
- Confirm data analysis capabilities using BigQuery to perform complex queries on stored data.

## Non-Functional Testing:
- **Performance Testing:** Measure crawl speed, data processing latency, and system response time under varying loads.
- **Scalability Testing:** Assess the system's ability to handle increased data volumes and concurrent crawl requests.
- **Security Testing:** Evaluate the effectiveness of security measures (e.g., Cloud Armor, authentication) in protecting against potential threats.

## Special Conditions:

- **Network Latency:** Simulate network delays to understand how crawl performance might be affected by different internet connection speeds.
- **Geographic Load Balancing:** Test the system's ability to distribute crawl tasks across geographically distributed servers for optimized performance.

# Rollout Plan: 

## Objective
Gradually release the Generic Web Crawler System to a wider audience while managing risks and ensuring smooth operation.

## Phases

### Phase 1: Development and Testing (Internal)
- **Develop Core Functionalities:**
  - Web crawlers, data extraction, storage systems.
- **Testing:**
  - Conduct thorough unit and integration testing.
- **Security Audits:**
  - Perform security audits to safeguard data and infrastructure.

### Phase 2: Alpha Release (Internal)
- **Deployment:**
  - Deploy the system to a controlled internal staging environment.
- **Internal User Testing:**
  - Invite a small group of internal users for testing.
- **Feedback and Refinement:**
  - Gather feedback on performance, usability, and identify critical bugs.
  - Refine the system based on internal testing results.

### Phase 3: Beta Release (External)
- **Limited Public Access:**
  - Open the system to a limited group of external beta testers.
- **Real-World Testing:**
  - Monitor system stability and performance under real-world conditions.
- **User Feedback:**
  - Collect user feedback on functionality and user experience.
- **Issue Resolution:**
  - Address emerging issues promptly.

### Phase 4: Staged Rollout (Production)
- **Initial Deployment:**
  - Release the system to a small percentage of the target user base.
- **Performance Monitoring:**
  - Closely monitor system performance metrics and logs.
- **Gradual Increase in Access:**
  - Gradually increase user access in stages based on successful performance.
- **Safety Measures:**
  - Revert to the previous stage if critical issues arise.

### Phase 5: Full Production Rollout
- **Full Deployment:**
  - Deploy the system across all operational regions after successful staged rollout.
- **Continuous Monitoring:**
  - Maintain continuous monitoring for unexpected behavior or potential problems.

## Post-Rollout Activities
- **Real-Time Monitoring:**
  - Implement real-time monitoring tools for system performance and error rates.
- **Automated Alerts:**
  - Set up automated alerts for critical incidents.
- **Continuous Improvement:**
  - Continuously improve the system with new features and address user feedback.

# Follow-Up Work

## Continuous Improvement:

- Implement data quality checks to monitor the accuracy and completeness of extracted data.
- Regularly analyze website access logs and crawl patterns to identify opportunities for improving crawling efficiency and cost optimization.
- Update crawling configurations based on website structure changes to ensure consistent data collection.

## Maintenance:

- Schedule regular updates for the web crawler software and Cloud Functions to benefit from security patches, bug fixes, and new features.
- Stay updated on website changes and adapt crawling configurations as needed to maintain functionality.
- Implement a version control system for the web crawler codebase to track changes and facilitate rollbacks if necessary.

