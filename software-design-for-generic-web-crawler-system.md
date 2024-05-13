# Software Design Document for Generic Web Crawler System (Public Version)

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

## Why We Are Using This Solution

- **Efficient Data Collection:** Standardizing the collection process ensures consistent, reliable data acquisition across multiple sources.
- **Cost Reduction:** Managing separate crawlers for individual websites is costly and difficult to maintain.
- **Scalability:** The system is designed to be flexible and scalable for current and future needs.
- **Security:** Implementing cloud-based security measures helps safeguard collected data.

# System Architecture

## Background and Motivation

Modern data collection frameworks struggle to efficiently handle the vast amounts of diverse data available from the web. This leads to:

- **Scalability Challenges:** Difficulty in scaling individual crawlers to accommodate increasing data volumes and website additions.
- **High Maintenance Overhead:** Managing numerous crawlers with different configurations becomes complex and time-consuming.
- **Limited Data Coverage:** Inconsistent crawling strategies might miss valuable data sources or fail to adapt to evolving website structures.

The Generic Web Crawler System aims to address these challenges by offering a standardized, scalable, and extensible solution for web data collection.

## Implementation Details

The system leverages Google Cloud Platform (GCP) for a distributed and highly available architecture. Here's a breakdown of the data flow and component responsibilities:

### Development
- **Language & Tools:** Utilize Python with libraries such as Scrapy or BeautifulSoup to develop a reusable crawler script.
- **Features:**
  - **Politeness:** Adherence to `robots.txt` to prevent overloading websites.
  - **Error Handling:** Robust error management to handle crawl interruptions gracefully.
  - **Middleware:** Incorporate middleware for tasks like authentication and data extraction.

### Deployment
- **Containerization:** Use Docker containers to encapsulate the crawler code and dependencies.
  - **Isolated Environments:** Ensure that each container operates independently to eliminate conflicts.
  - **Deployment Ease:** Enable consistent deployments across various environments with minimal configuration changes.

### Management
- **Kubernetes Engine:** Employ Kubernetes to manage the lifecycle and scalability of the crawler containers.
  - **Scalability:** Easily add crawlers for new sites and scale horizontally during peak times.

## Cloud Functions (Serverless)
- **Functionality:** Real-time processing of raw data using event-driven cloud functions.
- **Tasks Include:**
  - **Data Transformation:** Convert raw data into usable formats.
  - **Data Normalization:** Standardize data formats across different sources.
  - **Event Triggers:** Notify system administrators of key events like crawl completions or errors through Pub/Sub.

## Data Storage (Hybrid Approach)
- **Structured Data Storage:** Use MySQL Database (Cloud SQL) for storing structured data such as URLs and page analytics.
- **Unstructured Data Storage:** Utilize MongoDB Atlas on GCP to handle unstructured data like web content and images.

## Data Orchestration (Pub/Sub)
- **Message Queuing:** Ensure smooth data flow and system component decoupling, allowing for independent scaling and efficient resource management.

## Data Analysis and Retrieval (BigQuery)
- **Analytics:** Leverage BigQuery for complex queries and deep analysis of stored data to generate insights and reports.

## Responsibilities of Major Components
- **Web Crawler:** Manages the downloading, parsing, and initial data handling.
- **Cloud Functions:** Processes and refines data, manages event-based tasks.
- **Database Management:** Handles data storage and retrieval operations.
- **Data Analysis:** Performs data analysis and generates actionable reports.

## Load Balancing
- **HTTP(S) Load Balancing:** Implement load balancing to distribute requests efficiently across multiple crawler containers.
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

