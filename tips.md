

# **The Software Engineer's Field Guide to AWS: Avoiding Pitfalls and Optimizing Costs**

## **1\. Introduction: Adopting a Cloud-First Mindset**

### **The Developer's Evolving Role**

The adoption of cloud computing, particularly with Amazon Web Services (AWS), represents a fundamental shift in how applications are built, deployed, and maintained. For software engineers, this evolution extends their responsibilities beyond writing code. Under the AWS Shared Responsibility Model, AWS manages the security *of* the cloud—the physical infrastructure, hardware, and the software that runs the core services. However, the customer, which includes the development team, is responsible for security, operational excellence, and cost efficiency *in* the cloud. This means that configuration choices, architectural patterns, and even lines of code have direct implications for the security posture, performance, and financial cost of the application. This guide serves as a field manual for engineers to navigate these new responsibilities, focusing on the most common AWS services encountered in modern development workflows.1

### **Three Pillars of Cloud Excellence**

To build successful, sustainable applications on AWS, engineers must internalize three core principles. These pillars form the foundation of a cloud-first mindset and are the recurring themes throughout this guide.

1. **Build Securely by Default:** Security in the cloud cannot be an afterthought or a final step before deployment. It must be a foundational component of the design and development process. Many of the most common and damaging pitfalls in AWS are security-related, often stemming from the neglect of basic principles like least-privilege access or encryption. From an improperly configured S3 bucket to an overly permissive IAM role, these vulnerabilities are preventable with proactive security measures.4  
2. **Design for Performance and Resilience:** Cloud environments are dynamic. Hardware can fail, networks can experience latency, and demand can spike unpredictably. A cloud-native application must be architected to not only withstand failure but to thrive in this environment. This requires moving away from the single-server, monolithic mindset and embracing distributed, resilient patterns. Designing for high availability across multiple Availability Zones (AZs) and implementing automated scaling are not advanced techniques but baseline requirements for robust applications.4  
3. **Maintain Cost-Consciousness:** In the cloud, every resource allocation, every hour an instance runs, and every gigabyte of data transferred has a direct and measurable financial impact. The ease of provisioning resources can lead to "bill shock" if costs are not managed proactively. Adopting a FinOps (Financial Operations) mindset means that engineers share responsibility for the cost implications of their technical decisions. This involves choosing the right services, right-sizing resources, and automating cost-control measures to ensure that the application delivers value efficiently.4

---

## **2\. Amazon EC2 (Elastic Compute Cloud): The Foundational Workhorse**

Amazon EC2 provides scalable virtual servers, known as instances, and is often the starting point for applications moving to the cloud. While foundational, its flexibility is a double-edged sword, presenting numerous pitfalls for the unwary.1

### **2.1. Common Pitfalls & Mitigation**

#### **Security: The Unlocked Front Door**

* **Overly Permissive Security Groups:** A security group acts as a virtual firewall for an EC2 instance. One of the most frequent and dangerous mistakes is configuring a rule that allows unrestricted inbound traffic from the entire internet (using CIDR block 0.0.0.0/0) on sensitive ports, such as SSH (port 22\) or RDP (port 3301). This configuration is akin to leaving the front door of a data center wide open, inviting automated brute-force attacks from malicious actors across the globe.4  
  * **Mitigation:** Apply the principle of least privilege. Security group rules should be as specific as possible. For administrative access like SSH, inbound rules should be restricted to a specific, known IP address or a corporate VPN's IP range. For application traffic, an instance's security group should only allow inbound connections from the security group of the upstream resource, such as a load balancer or another application tier.  
* **Mishandled Credentials and IAM Roles:** Hardcoding static AWS access keys and secret keys directly into application code or configuration files is a critical security vulnerability. If this code is ever committed to a public repository, these credentials can be scraped by automated bots within minutes, leading to a complete account compromise.4  
  * **Mitigation:** Never store static credentials on an EC2 instance. The correct and secure method is to use **IAM Roles for EC2**. An IAM Role is attached to the instance at launch, and it provides the application with temporary, automatically rotated credentials through the instance metadata service. This eliminates the risk of exposed long-lived keys and adheres to security best practices.2

#### **Performance: Mismatched Workloads and Resources**

* **Choosing the Wrong Instance Family:** AWS offers a wide array of EC2 instance families, each optimized for different types of workloads. Selecting a general-purpose instance (e.g., t3 or m5) for a compute-intensive data processing job, or a compute-optimized instance (e.g., c5) for a memory-intensive in-memory database, leads to either performance bottlenecks or wasted resources.4  
  * **Mitigation:** Characterize the application's workload before selecting an instance type.  
    * **General Purpose (M, T series):** Balanced CPU, memory, and networking. Good for web servers and small-to-medium databases.  
    * **Compute Optimized (C series):** High CPU-to-memory ratio. Ideal for batch processing, media transcoding, and high-performance computing.  
    * **Memory Optimized (R, X series):** High memory-to-CPU ratio. Best for in-memory databases, real-time big data analytics, and other memory-intensive applications.  
* **EBS I/O Bottlenecks:** The performance of an EC2 instance is not solely determined by its CPU and memory. The attached Elastic Block Store (EBS) volume, which acts as the instance's hard drive, has its own performance characteristics for Input/Output Operations Per Second (IOPS) and throughput. Developers often overlook that the performance of gp2 volumes is tied to their size, and an under-provisioned volume can become an I/O bottleneck, starving the application and causing severe performance degradation even if the EC2 instance itself is oversized.10  
  * **Mitigation:** Monitor EBS-specific metrics in Amazon CloudWatch, such as VolumeReadOps, VolumeWriteOps, and VolumeQueueLength. A consistently high queue length is a strong indicator of an I/O bottleneck. For new workloads, it is highly recommended to use gp3 volumes. Unlike gp2, gp3 volumes allow for the independent provisioning of storage size, IOPS, and throughput, providing better performance and often at a lower cost.

#### **Architecture: The Monolith in the Cloud**

* **Neglecting Auto Scaling:** A common anti-pattern inherited from on-premises environments is to provision a small number of very large EC2 instances designed to handle peak traffic. This approach is both expensive and inefficient, as the excess capacity sits idle during non-peak hours. It also lacks resilience; if one of the large instances fails, a significant portion of the application's capacity is lost.4  
  * **Mitigation:** Embrace elasticity by using an **Amazon EC2 Auto Scaling group**. Auto Scaling allows the application to dynamically adjust the number of EC2 instances in response to real-time demand. By setting scaling policies based on metrics like average CPU utilization or request count, the system can automatically launch new instances during traffic spikes and terminate them when demand subsides, ensuring optimal performance at the lowest possible cost.12  
* **Failing to Design for Multi-AZ Resilience:** An Availability Zone (AZ) is a distinct data center within an AWS Region. Deploying a critical application's instances in a single AZ creates a single point of failure. Any issue affecting that specific AZ—such as a power outage or network failure—will result in a complete application outage.4  
  * **Mitigation:** Architect for high availability by configuring the Auto Scaling group to launch instances across multiple (typically two or three) Availability Zones. An Elastic Load Balancer (ELB) should be placed in front of the Auto Scaling group to distribute incoming traffic evenly across the instances in all active AZs. If an instance or an entire AZ fails, the ELB will automatically reroute traffic to the remaining healthy instances, providing seamless failover and maintaining application availability.13

The choice of an EC2 instance is not an isolated decision; it has cascading effects on performance, cost, and security. A developer might select a small, general-purpose instance to minimize initial costs. However, under load, this instance could suffer from high CPU utilization, leading to slow response times.4 The typical reaction is to "scale up" to a much larger and more expensive instance. This reactive right-sizing might not even solve the problem if the bottleneck was actually I/O-related, which a larger instance of the same family might not address.11 This scenario reveals a deeper truth: reactive scaling is often a symptom of poor initial architectural choices. Proactively characterizing the application's workload and selecting the appropriate instance family and storage configuration from the outset prevents these costly and disruptive fire drills.

#### **Diagram: High-Availability EC2 Architecture**

The following diagram illustrates a resilient and scalable EC2 architecture. An Application Load Balancer distributes traffic across two Availability Zones. Within each AZ, an Auto Scaling group manages EC2 instances in a private subnet. These instances use a NAT Gateway in a public subnet for outbound internet access. This design ensures high availability, security by isolation, and cost-efficiency through dynamic scaling.

\!(https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)  
Source: 16

### **2.2. Cost Optimization Strategies**

#### **Right-Sizing: Eliminating Waste**

Right-sizing is the process of matching instance types and sizes to workload performance and capacity requirements at the lowest possible cost. A significant source of wasted cloud spend comes from over-provisioned instances.

* **Strategy:** Regularly use AWS-native tools to identify optimization opportunities.  
  * **AWS Cost Explorer Resource Optimization:** This tool provides reports on idle or underutilized EC2 instances with specific recommendations to downsize or terminate them.8  
  * **AWS Compute Optimizer:** This service analyzes configuration and utilization metrics to recommend optimal AWS resources. It can suggest downsizing instances within the same family or even recommend switching to a different, more cost-effective family.17

#### **Pricing Models: The Biggest Lever for Savings**

Relying solely on the On-Demand pricing model is one of the most expensive ways to run sustained workloads on AWS. Understanding and utilizing the different purchasing options is critical for significant cost savings.8

* **On-Demand:** This is the default pay-as-you-go model with no long-term commitment. It offers the most flexibility but comes at the highest price. It is ideal for short-term, spiky, or unpredictable workloads, and for initial application development and testing.  
* **Savings Plans:** These offer a significant discount (up to 72%) in exchange for a commitment to a consistent amount of compute usage (measured in $/hour) for a 1 or 3-year term. This is the most flexible discount model and is highly recommended for any application with a predictable, steady-state workload.  
* **Reserved Instances (RIs):** Similar to Savings Plans, RIs provide a large discount for a 1 or 3-year commitment. However, they are less flexible, as they reserve capacity for a specific instance family in a specific AWS Region. They are best suited for workloads that will remain static for the entire term.  
* **Spot Instances:** This model allows you to bid on spare, unused EC2 capacity at discounts of up to 90% off the On-Demand price. The trade-off is that AWS can reclaim these instances with only a two-minute warning. Spot Instances are an excellent choice for fault-tolerant, stateless, or batch-processing workloads, such as big data analysis, CI/CD pipelines, and high-performance computing, where interruptions can be gracefully handled.8

#### **Automation: Your Cost-Cutting Co-pilot**

* **Scheduled Shutdowns:** A major source of unnecessary cost is leaving non-production environments (development, testing, staging) running 24/7. These resources are typically only needed during business hours.  
  * **Strategy:** Use the **AWS Instance Scheduler**, a solution that automates the starting and stopping of EC2 and RDS instances based on a predefined schedule. Implementing an "office hours" schedule can reduce the cost of these environments by approximately 70%.8  
* **Orphaned Resource Cleanup:** When an EC2 instance is terminated, its associated resources, such as EBS volumes or Elastic IP addresses, may not be deleted automatically. These "orphaned" resources continue to incur charges while providing no value.4  
  * **Strategy:** When launching an instance, ensure the "Delete on Termination" flag is checked for its EBS volumes. Additionally, implement automated scripts using AWS Lambda and CloudWatch Events to periodically scan for and report on unattached EBS volumes and unassociated Elastic IPs, allowing for their safe removal.

#### **Table: EC2 Purchasing Options Comparison**

| Pricing Model | Commitment | Discount vs. On-Demand | Ideal Workload | Key Consideration |  |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **On-Demand** | None | 0% | Short-term, unpredictable, development/testing | Highest cost but maximum flexibility. |  |
| **Savings Plans** | 1 or 3-year ($/hour) | Up to 72% | Predictable, steady-state usage | Flexible across instance families and regions. |  |
| **Reserved Instances** | 1 or 3-year (instance) | Up to 75% | Stable, static workloads | Locks you into a specific instance family and region. |  |
| **Spot Instances** | None | Up to 90% | Fault-tolerant, stateless, batch jobs | Instances can be terminated with a 2-minute warning. |  |
| Source: 8 |  |  |  |  |  |

---

## **3\. Amazon S3 (Simple Storage Service): Scalable Object Storage**

Amazon S3 is a highly scalable, durable, and secure object storage service. It serves as the backbone for a vast number of applications, from data lakes and backup archives to static website hosting and content delivery.1

### **3.1. Common Pitfalls & Mitigation**

#### **Security: The Publicly Leaked Bucket**

* **Public Buckets by Default:** The most infamous S3 pitfall is accidentally configuring a bucket to be publicly accessible, exposing all of its data to the open internet. This has been the root cause of numerous high-profile data breaches.5  
  * **Mitigation:** The non-negotiable first line of defense is to enable **S3 Block Public Access** at the AWS account level. This feature provides a centralized control to override any bucket-level policies or ACLs that would otherwise allow public access, acting as a critical safety net against accidental exposure.5  
* **Misconfigured Bucket Policies & ACLs:** Even if a bucket is not fully public, poorly configured bucket policies or Access Control Lists (ACLs) can grant overly permissive access to unauthorized users or services. Understanding the hierarchy of permissions (IAM policies, bucket policies, ACLs) is crucial to avoid unintended data exposure.5  
  * **Mitigation:** Adhere strictly to the principle of least privilege. Use IAM roles and policies as the primary mechanism for granting access to S3 resources. Bucket policies should be used for explicit cross-account access or to enforce specific conditions. ACLs are a legacy mechanism and should generally be disabled in favor of IAM and bucket policies.  
* **Neglecting Encryption:** Storing sensitive, unencrypted data in S3 is a significant security and compliance risk. In the event of a misconfiguration or credential compromise, unencrypted data is immediately readable by an attacker.19  
  * **Mitigation:** Enable default encryption on all S3 buckets. AWS offers several server-side encryption (SSE) options:  
    * **SSE-S3:** AWS manages the encryption keys using AES-256. This is the simplest option.  
    * SSE-KMS: AWS Key Management Service (KMS) manages the encryption keys. This provides an audit trail of key usage and allows for customer-managed keys (CMKs) for greater control.  
      Enabling default encryption ensures that all new objects uploaded to the bucket are automatically encrypted at rest without any action required by the uploading client.19

#### **Data Management: The Digital Hoarder**

* **Forgetting to Enable Versioning:** S3 versioning is a powerful feature that keeps a history of all versions of an object. It acts as a safeguard against accidental deletions or overwrites. If versioning is not enabled, an accidental DELETE operation is permanent and the data is irrecoverable.19  
  * **Mitigation:** Enable versioning on all buckets containing critical or production data. This allows for the easy recovery of objects that have been unintentionally deleted or overwritten. It is important to pair versioning with a lifecycle policy to manage the storage costs of noncurrent versions.  
* **Ignoring Logging and Monitoring:** Without proper logging, it is impossible to audit who is accessing your data, from where, and when. In the event of a suspected security incident, the absence of logs makes investigation and response incredibly difficult, if not impossible.5  
  * **Mitigation:** Enable **S3 server access logging** and **AWS CloudTrail data events** for S3. Server access logs provide detailed records for every request made to a bucket. CloudTrail provides an audit trail of API calls made to S3. Logs should be directed to a separate, dedicated, and highly secured S3 bucket.

The operational model of S3 creates a subtle but dangerous disconnect between an action and its long-term consequences. A developer can upload a large file in seconds, a seemingly trivial act. However, if that bucket lacks a lifecycle policy, the file will remain in the most expensive storage tier indefinitely.19 If versioning is also enabled without a corresponding cleanup policy, every overwrite of that file creates a new copy, silently doubling the storage cost.19 Over months, the cumulative effect of this inaction across thousands of files can lead to a surprisingly large S3 bill. This illustrates that the most significant S3 pitfall is not a single misconfiguration but the systemic failure to implement automated governance from the very beginning.

### **3.2. Cost Optimization Strategies**

#### **Storage Tiers: Matching Cost to Access Patterns**

Storing all data in the default S3 Standard storage class is a common and costly mistake. S3 offers a range of storage classes designed to provide the lowest cost for different data access patterns.1

* **S3 Standard:** Designed for frequently accessed, "hot" data that requires low latency and high throughput. It is the most expensive storage tier.  
* **S3 Intelligent-Tiering:** This should be the default choice for data with unknown, changing, or unpredictable access patterns. It automatically monitors access patterns and moves objects between a frequent access tier and an infrequent access tier to optimize costs. It does this with no performance impact, no retrieval fees, and no operational overhead, making it a powerful tool for automatic cost savings.1  
* **S3 Standard-Infrequent Access (S3 Standard-IA) & S3 One Zone-IA:** These are for long-lived, but less frequently accessed data that still requires rapid access when needed (e.g., backups, disaster recovery files). They offer a lower storage price than S3 Standard but charge a per-GB retrieval fee. One Zone-IA is cheaper as it stores data in a single AZ, making it suitable for recreatable data.  
* **S3 Glacier Storage Classes (Instant Retrieval, Flexible Retrieval, Deep Archive):** These are designed for long-term data archiving at extremely low storage costs. They differ by retrieval time and cost, ranging from milliseconds (Glacier Instant Retrieval) to hours (Glacier Deep Archive). These tiers are ideal for compliance archives and data that is rarely, if ever, accessed.

#### **Lifecycle Policies: Automate Everything**

S3 Lifecycle policies are the cornerstone of S3 cost management. They are rules that automate the management of an object's lifecycle, transitioning it to cheaper storage tiers over time and eventually deleting it when it is no longer needed.21

* **Strategy:** Create lifecycle policies for all buckets. A typical policy might:  
  1. Transition objects from S3 Standard to S3 Intelligent-Tiering or Standard-IA after 30-60 days.  
  2. Transition objects to S3 Glacier Deep Archive after 180 days for long-term retention.  
  3. Permanently delete objects after a defined compliance period (e.g., 7 years).  
  4. Clean up expired object delete markers and incomplete multipart uploads.  
  5. Expire noncurrent (old) versions of objects after a short period (e.g., 30 days) to save on storage costs in versioned buckets.

#### **Example: S3 Lifecycle Policy JSON**

The following JSON configuration defines a lifecycle policy for a bucket. It transitions objects with the logs/ prefix through various storage classes and eventually expires them. It also cleans up incomplete multipart uploads and old object versions for the entire bucket.

```JSON
{  
    "Rules":,  
            "Expiration": {  
                "Days": 2555  
            }  
        },  
        {  
            "ID": "Abort-Incomplete-Multipart-Uploads",  
            "Filter": {},  
            "Status": "Enabled",  
            "AbortIncompleteMultipartUpload": {  
                "DaysAfterInitiation": 7  
            }  
        },  
        {  
            "ID": "Expire-Noncurrent-Versions",  
            "Filter": {},  
            "Status": "Enabled",  
            "NoncurrentVersionExpiration": {  
                "NoncurrentDays": 30  
            }  
        }  
    \]  
}
```

Source: 24

#### **Data Transfer: The Hidden Cost**

Data transfer *out* from S3 to the public internet is a significant cost factor that often surprises development teams.19 Serving popular content, such as images or videos, directly from S3 can lead to substantial data transfer charges.

* **Strategy: Use Amazon CloudFront:** CloudFront is AWS's Content Delivery Network (CDN). By configuring an S3 bucket as an origin for a CloudFront distribution, the content is cached at AWS's global network of edge locations. When a user requests the content, it is served from the nearest edge location, which dramatically improves latency. Crucially, data transfer from S3 to CloudFront is free. This means you only pay for the much lower CloudFront data transfer rates, significantly reducing overall costs for content delivery workloads.19

#### **Table: S3 Storage Class Comparison**

| Storage Class | Designed For | Retrieval Time | Min. Storage Duration | Retrieval Fee |  |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **S3 Standard** | Frequently accessed data | Milliseconds | None | No |  |
| **S3 Intelligent-Tiering** | Unknown or changing access | Milliseconds | None | No |  |
| **S3 Standard-IA** | Infrequently accessed data | Milliseconds | 30 days | Yes |  |
| **S3 One Zone-IA** | Recreatable, infrequent data | Milliseconds | 30 days | Yes |  |
| **S3 Glacier Instant Retrieval** | Archive data, instant access | Milliseconds | 90 days | Yes |  |
| **S3 Glacier Flexible Retrieval** | Archive data, flexible access | Minutes to Hours | 90 days | Yes |  |
| **S3 Glacier Deep Archive** | Long-term archive | Hours | 180 days | Yes |  |
| Source: 1 |  |  |  |  |  |

---

## **4\. AWS Lambda: The Core of Serverless Computing**

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers. It executes code in response to events and automatically manages the underlying compute resources, making it a cornerstone of modern, event-driven architectures.1

### **4.1. Common Pitfalls & Mitigation**

#### **Performance: The Serverless Slowdown**

* **Cold Starts:** A "cold start" is the latency incurred on the first invocation of a Lambda function after a period of inactivity. During this time, Lambda must provision a new execution environment, download the code, and initialize the runtime. This can add hundreds of milliseconds or even seconds to the invocation time, which can be unacceptable for latency-sensitive, user-facing APIs.27  
  * **Mitigation:**  
    1. **Provisioned Concurrency:** For critical functions, configure Provisioned Concurrency to keep a specified number of execution environments "warm" and ready to respond instantly, eliminating cold starts at the cost of an hourly fee.28  
    2. **Minimize Package Size:** Keep deployment packages as small as possible by including only necessary dependencies. Smaller packages load faster.  
    3. **Avoid VPCs (When Possible):** Placing a Lambda function inside a VPC can significantly increase cold start times due to the need to create and attach an Elastic Network Interface (ENI). Only place functions in a VPC if they absolutely require access to private resources like an RDS database.27  
* **Inefficient Function Code:** Lambda pricing is a function of execution duration and memory allocation. Inefficient code—such as synchronous I/O calls, complex computations in the main handler, or repeatedly initializing SDK clients on every invocation—directly increases execution time and, therefore, cost.27  
  * **Mitigation:** Write small, single-purpose functions. Use asynchronous patterns to handle I/O operations without blocking. Initialize database connections and SDK clients outside of the main handler function so they can be reused across invocations within the same execution environment.

#### **Configuration: The Devil is in the Details**

* **Incorrect Memory and Timeout Settings:** Developers often make one of two mistakes: they either over-provision memory and set excessively long timeouts "just in case," which inflates costs for no benefit, or they under-provision, causing functions to fail with out-of-memory errors or timeouts.27  
  * **Mitigation:** Right-size every function. Use monitoring and tools like AWS Lambda Power Tuning to find the optimal memory setting that provides the best price-to-performance ratio. Set the timeout to a reasonable value (e.g., 2-3 times the average execution duration) to prevent runaway functions from incurring excessive costs.  
* **Neglecting Concurrency Controls:** By default, an AWS account has a limit on the number of concurrent Lambda executions (typically 1,000 per region). A sudden traffic spike or a misbehaving function that triggers a recursive loop can rapidly consume this entire limit. This causes all other Lambda functions in the account to be throttled, creating a cascading failure across multiple applications.27  
  * **Mitigation:** Use **Reserved Concurrency**. By reserving a specific amount of concurrency for a critical function, you both guarantee that it can scale to that level and, just as importantly, you cap its maximum concurrency, preventing it from overwhelming the account-wide limit and impacting other functions.

#### **Architecture: The "Lambda-lith"**

* **The Monolithic Function:** A common anti-pattern is to take a large, monolithic application and attempt to port it into a single, massive Lambda function. This approach negates nearly all the benefits of serverless, resulting in a function that is slow to start, difficult to debug, and impossible to scale granularly.27  
  * **Mitigation:** Decompose the application into a set of small, loosely coupled, single-purpose functions. This aligns with a microservices architecture, where each function is responsible for one business capability and is triggered by a specific event (e.g., an API call, a message in a queue).  
* **Ignoring Dead-Letter Queues (DLQs):** When a Lambda function is invoked asynchronously (e.g., by an S3 event or an SNS notification) and fails after all retries are exhausted, the triggering event is discarded and lost forever. This can lead to silent data loss.27  
  * **Mitigation:** Always configure a Dead-Letter Queue (DLQ) for asynchronous Lambda functions. A DLQ is an SQS queue or SNS topic where failed events are sent. This captures the failed invocation data, allowing developers to inspect the payload, diagnose the root cause of the failure, and replay the event after the issue is fixed.

In the Lambda pricing model, memory allocation is directly coupled with the amount of vCPU power a function receives. This creates a non-obvious optimization path. A developer might have a CPU-bound function running at the lowest memory setting (128MB) to minimize the cost-per-millisecond.30 If this function takes 2 seconds to run, increasing the memory to 256MB might seem counter-intuitive as it doubles the per-millisecond cost. However, because this also provides more CPU power, the function might now execute in just 0.8 seconds. Even though the rate is higher, the duration is reduced by more than half, resulting in a lower overall invocation cost (

memory \* duration). This demonstrates a critical principle of Lambda optimization: the goal is not always to minimize memory, but to find the point on the price-performance curve where the total invocation cost is lowest. This requires deliberate testing, not guesswork.

#### **Diagram: Serverless Web Application Architecture**

This diagram shows a typical serverless pattern. A user request is routed through Amazon Route 53 to an API Gateway endpoint. The API Gateway invokes a Lambda function to handle the business logic. The Lambda function interacts with a DynamoDB table for data persistence and may send a message to an SQS queue to trigger a separate, asynchronous Lambda function for background processing.

\!(https://d1.awsstatic.com/whitepapers/serverless-multi-tier-architectures-api-gateway-lambda/web-application.png)  
Source: 32

### **4.2. Cost Optimization Strategies**

#### **Right-Sizing Memory**

This is the most impactful lever for controlling Lambda costs. Since memory allocation also determines CPU power, finding the optimal setting is crucial.

* **Strategy:** Use the **AWS Lambda Power Tuning** tool. This open-source state machine, deployed via the AWS Serverless Application Repository, automates the process of running a function at various memory configurations. It measures the performance and cost of each configuration and generates a visualization to help you identify the setting that offers the best balance of cost and speed for your specific workload.29

#### **Choose the Right Processor Architecture**

AWS Lambda functions can run on either traditional x86 processors or AWS's custom ARM-based Graviton2 processors.

* **Strategy:** For many workloads, functions running on the **ARM/Graviton2** architecture can provide up to 20% better price-performance compared to x86.29 When starting a new project, default to the ARM architecture unless a specific third-party dependency or library requires x86. The performance gains and cost savings are often significant.

#### **Optimize Code and Dependencies**

* **Strategy:** Faster code is cheaper code.  
  * **Connection Re-use:** In your function code, initialize SDK clients and database connections *outside* of the main handler function. This allows them to be persisted and reused across multiple invocations within the same warm execution environment, avoiding the latency and overhead of re-establishing connections on every request.  
  * **Lambda Layers:** If you have common dependencies (like SDKs or helper libraries) shared across multiple Lambda functions, package them into a **Lambda Layer**. This reduces the size of your individual function deployment packages, which can lead to faster cold starts and simplifies dependency management.30

#### **Leverage Event Batching**

For Lambda functions triggered by streaming services like Amazon SQS and Kinesis, you can process events in batches.

* **Strategy:** Configure a BatchSize and BatchWindow. This allows a single Lambda invocation to receive and process multiple messages or records (up to the configured batch size) from the queue or stream. This dramatically reduces the total number of invocations, which is a primary component of Lambda pricing. For high-volume, event-driven workloads, batching is one of the most effective cost-saving techniques.29

---

## **5\. Amazon RDS (Relational Database Service): Managed Relational Databases**

Amazon RDS simplifies the setup, operation, and scaling of relational databases in the cloud. It manages time-consuming administration tasks like hardware provisioning, patching, and backups, allowing developers to focus on their applications.7

### **5.1. Common Pitfalls & Mitigation**

#### **Connectivity & Security**

* **Misconfigured Security Groups:** A frequent and critical error is exposing an RDS instance to the public internet by allowing inbound traffic from 0.0.0.0/0 on the database port (e.g., 3306 for MySQL, 5432 for PostgreSQL). This makes the database a direct target for attacks.7  
  * **Mitigation:** An RDS instance should almost always reside in a private subnet. Its security group should be configured to allow inbound traffic *only* from the security group of the application servers that need to connect to it. This creates a tightly controlled, multi-layered security perimeter.  
* **Neglecting Encryption:** Storing sensitive customer or business data in an unencrypted database is a major security and compliance violation. If the underlying storage is compromised, the data is freely accessible.6  
  * **Mitigation:** Always enable **Encryption at Rest** when creating a new RDS instance. This is a simple checkbox that uses AWS KMS to encrypt the underlying storage volumes and backups. Additionally, enforce **Encryption in Transit** by requiring SSL/TLS connections from your application, which protects data as it travels over the network between the application server and the database.

#### **Performance**

* **Under-provisioned Instances:** Choosing a small, burstable instance class (like db.t3.micro) for a production workload is a recipe for poor performance. These instances rely on a CPU credit balance, which can be quickly exhausted under sustained load, leading to the instance being throttled to a very low baseline performance level.7  
  * **Mitigation:** Continuously monitor key performance metrics in Amazon CloudWatch, such as CPUUtilization, FreeableMemory, and Read/WriteIOPS. For deeper analysis, use **Amazon RDS Performance Insights**, a tool that provides a visual dashboard to help identify and diagnose performance bottlenecks, such as slow-running queries.7 Use this data to select an appropriately sized instance from a fixed-performance instance class (e.g.,  
    db.m5 or db.r5) for production workloads.  
* **Ignoring Read Replicas:** A common architectural flaw is directing all database traffic—both writes and reads—to a single primary database instance. Read-heavy applications, especially those with analytical dashboards or reporting features, can easily overwhelm the primary instance, degrading performance for all users.4  
  * **Mitigation:** Offload read traffic by creating one or more **Read Replicas**. A read replica is a read-only copy of the primary database that is kept up-to-date via asynchronous replication. The application can be configured to send all write operations (INSERT, UPDATE, DELETE) to the primary instance's endpoint and all read operations (SELECT) to the read replica's endpoint. This effectively scales the read capacity of the database tier.36

#### **Operations**

* **Inadequate Backup Strategy:** While RDS provides automated daily backups by default, relying solely on these may not be sufficient. The default retention period might be too short for compliance requirements, and in the case of a critical failure, the recovery process might take longer than the business's Recovery Time Objective (RTO) allows.7  
  * **Mitigation:** Customize the automated backup window and retention period to meet your application's specific Recovery Point Objective (RPO). For critical operations, take **manual snapshots** before performing major application upgrades or schema changes. These snapshots are independent of the automated backups and are retained until you explicitly delete them, providing an extra layer of protection.

The "managed" nature of RDS can create a false sense of security, leading developers to treat the database as an opaque black box. This often results in a pattern where any performance issue is reflexively blamed on the infrastructure. For example, when an application slows down, a developer might observe high CPUUtilization in CloudWatch and conclude that the RDS instance is underpowered.7 Their immediate request is to scale up the instance, for example from a

db.m5.large to a db.m5.xlarge, which instantly doubles the cost.38 While this might temporarily alleviate the symptom, a deeper investigation with Performance Insights would likely have revealed the true culprit: an inefficient, unindexed query in the application code causing full table scans.7 The root cause was not the instance size but an application-level inefficiency. This highlights a critical lesson: in a managed service like RDS, developers must use the provided observability tools to diagnose problems correctly before resorting to expensive and often ineffective infrastructure scaling.

#### **Diagram: RDS Architecture with Read Replica**

This diagram illustrates a highly available and scalable RDS deployment. The application's EC2 instances send write operations to a primary RDS instance in Availability Zone A. A Read Replica is provisioned in Availability Zone B, receiving updates via asynchronous replication. The application directs read-intensive queries to the Read Replica, scaling read capacity and isolating analytical workloads from the primary transactional database.

\!(https://d1.awsstatic.com/product-marketing/RDS/read-replicas.88a73c7d633391d7524a21059f23908f09b555e5.png)  
Source: 36

### **5.2. Cost Optimization Strategies**

#### **Right-Sizing Instances**

Just like with EC2, over-provisioning is a major source of wasted spend in RDS.

* **Strategy:** Implement a policy of continuous right-sizing. Use CloudWatch and Performance Insights to monitor database utilization. A common policy is to flag any production instance with CPU and I/O utilization consistently below 30% as a candidate for downsizing. For non-production environments, this threshold can be more aggressive, such as 50%.38

#### **Storage Optimization**

* **GP2 vs. GP3:** For database engines that support it, migrate RDS instances from older gp2 EBS volumes to gp3. gp3 volumes offer predictable baseline performance and allow you to provision IOPS and throughput independently of storage size. This often results in better performance at a lower or equivalent cost compared to a gp2 volume of the same size.38  
* **Deleting Unused Snapshots:** Manual RDS snapshots are not deleted automatically as part of the backup retention policy. Over time, these snapshots can accumulate and consume a significant amount of storage, leading to unexpected costs.  
  * **Strategy:** Implement a regular audit process, either manual or automated, to identify and delete manual snapshots that are no longer required for compliance or recovery purposes.38

#### **Instance Scheduling**

Non-production RDS instances used for development, testing, or staging are often only needed during business hours.

* **Strategy:** Use the AWS Instance Scheduler or custom Lambda scripts to automatically stop these instances in the evenings and on weekends. While the instance is stopped, you are billed for its provisioned storage but not for the instance hours, which can lead to savings of over 60% on those resources.9

#### **Reserved Instances**

For production databases with stable, predictable workloads, Reserved Instances (RIs) are the most effective cost-saving tool.

* **Strategy:** Once an application is in a steady state, purchase RDS Reserved Instances for a 1 or 3-year term. This commitment can provide discounts of up to 75% compared to the On-Demand price, dramatically reducing the cost of your most critical database workloads.8

---

## **6\. Amazon DynamoDB: Managed NoSQL Database**

Amazon DynamoDB is a fully managed, key-value and document NoSQL database that delivers single-digit millisecond performance at any scale. Its serverless nature and scalability make it a popular choice for modern applications.2

### **6.1. Common Pitfalls & Mitigation**

#### **Data Modeling: The Relational Mindset Trap**

* **Not Designing for Access Patterns:** The single most critical pitfall in DynamoDB is attempting to design a schema with a relational mindset. DynamoDB does not support server-side joins or complex, ad-hoc queries. Therefore, the data model must be designed from the ground up to satisfy the specific access patterns of the application. The goal is to design a table where all the data required for a given query can be fetched in a single, efficient request.39  
  * **Mitigation:** Before writing any code, exhaustively list every read and write operation the application will need to perform. Then, design a single table (a common best practice) with a primary key and secondary indexes that can satisfy all of these patterns efficiently. This upfront design effort is non-negotiable for a successful DynamoDB implementation.  
* **Overusing Scans:** A Scan operation in DynamoDB reads every single item in a table and then filters the results. At scale, this is incredibly slow, consumes a vast amount of read capacity, and is prohibitively expensive. A Scan operation in a production application is almost always a sign of a data modeling flaw.41  
  * **Mitigation:** Design your table so that all access patterns can be served by GetItem (which fetches a single item by its full primary key) or Query (which fetches a collection of items that share the same partition key). If you find yourself needing to perform a Scan, it is a strong signal that you need to revisit your data model or add a Global Secondary Index (GSI) to support that access pattern.

#### **Performance: The Hot Partition Problem**

* **Poor Partition Key Design:** DynamoDB achieves its scalability by distributing data across multiple underlying storage partitions based on the partition key. A "hot partition" occurs when a disproportionate amount of read or write traffic is directed to a single partition, overwhelming its fixed throughput limits and causing requests to be throttled (rejected with a ProvisionedThroughputExceededException).42 This is most often caused by choosing a partition key with low cardinality—that is, a key with only a few distinct values.  
  * **Mitigation:** The cornerstone of avoiding hot partitions is to choose a partition key with high cardinality that distributes requests as evenly as possible across all partitions.  
  * **Example: Good vs. Bad Partition Key**  
    * **Use Case:** An e-commerce application storing order data.  
    * **Bad Design:** Using PK: order\_status. Since there are only a few statuses (e.g., "PENDING", "SHIPPED", "DELIVERED"), all orders with the same status would be written to the same partition, creating a massive hot spot for the "PENDING" status.  
    * **Good Design:** Using PK: customer\_id. This key has very high cardinality, ensuring that orders from different customers are spread across many different partitions. To retrieve all orders for a specific customer, a Query operation on their customer\_id would be highly efficient.44 For extremely high write throughput scenarios, a technique called "write sharding" can be used, where a random number suffix is appended to the partition key to further distribute writes.

The strict requirement to "know your access patterns" before designing a DynamoDB table can initially seem like a frustrating limitation, especially for developers accustomed to the flexibility of SQL. However, this constraint is actually a hidden benefit that enforces superior application architecture. In the relational world, the flexibility of JOINs and ad-hoc queries often allows developers to defer hard decisions about data access, leading to inefficient queries and performance problems that are only discovered late in the development cycle.39 DynamoDB's model, in contrast, forces a deliberate, upfront conversation between developers and architects about exactly how the application will interact with its data. This process compels a level of design discipline that prevents the "we'll fix it in the query later" mentality. The result is an application whose data access paths are planned and optimized from day one, leading to a system that is inherently more scalable, performant, and predictable.

### **6.2. Cost Optimization Strategies**

#### **Capacity Modes: On-Demand vs. Provisioned**

This is the primary cost decision for any DynamoDB table and depends entirely on the workload's traffic pattern.41

* **On-Demand:** With this mode, you pay per request for the reads and writes your application performs. There is no capacity planning required; the table scales automatically to handle the traffic. This mode is ideal for new applications with unknown traffic patterns, applications with unpredictable or spiky workloads, or in development and testing environments. While convenient, it can be more expensive than Provisioned mode for sustained, high-volume traffic.  
* **Provisioned:** In this mode, you specify the number of Read Capacity Units (RCUs) and Write Capacity Units (WCUs) your application requires per second. You pay for this provisioned capacity, regardless of whether you consume it. This mode is more cost-effective for applications with predictable and consistent traffic patterns, as the per-request cost is lower than On-Demand.

#### **Auto Scaling**

For tables using Provisioned capacity mode, manual management can be challenging. Over-provisioning leads to wasted cost, while under-provisioning leads to throttling and a poor user experience.

* **Strategy:** Enable **DynamoDB Auto Scaling**. This feature automatically monitors the actual consumed capacity of your table and adjusts the provisioned RCUs and WCUs up or down in response to traffic changes. It helps maintain performance while protecting against over-provisioning, striking an effective balance between cost and availability.41

#### **Table Class**

For certain use cases, the storage cost of a DynamoDB table can become significant.

* **Strategy:** For tables that store long-lived data that is infrequently accessed, consider using the **DynamoDB Standard-Infrequent Access (Standard-IA)** table class. This class offers up to a 60% reduction in storage costs compared to the standard table class. The trade-off is that the per-request cost for reads and writes is higher. This makes it a good fit for use cases like storing historical user data or order archives that are rarely queried but must be retained.41

#### **Time to Live (TTL)**

Many applications generate transient data, such as session information, temporary logs, or event data, that only needs to be stored for a limited time.

* **Strategy:** Use **DynamoDB Time to Live (TTL)**. By enabling TTL on a table and specifying a timestamp attribute on your items, you can instruct DynamoDB to automatically delete items once their timestamp expires. This is a free feature that provides a powerful mechanism for managing data retention and preventing tables from growing indefinitely, which saves on storage costs and keeps table scans (if any are used) more performant.42

#### **Table: DynamoDB Capacity Mode Comparison**

| Feature | On-Demand Capacity | Provisioned Capacity |  |
| :---- | :---- | :---- | :---- |
| **Pricing Model** | Pay-per-request | Pay for provisioned throughput ($/hour) |  |
| **Best For** | Unpredictable, spiky, or new workloads | Predictable, consistent workloads |  |
| **Performance** | Scales instantly to handle traffic spikes | Throttles if traffic exceeds provisioned capacity |  |
| **Management Overhead** | Zero; fully automatic | Requires monitoring and Auto Scaling configuration |  |
| **Cost Predictability** | Can be variable and hard to forecast | Highly predictable and budgetable |  |
| Source: 46 |  |  |  |

---

## **7\. Amazon VPC (Virtual Private Cloud): Networking Foundations**

Amazon VPC lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. It is the foundational networking layer for nearly all AWS services.2

### **7.1. Common Pitfalls & Mitigation**

#### **Design: Painting Yourself into a Corner**

* **Insufficient CIDR Range:** When creating a VPC, you must specify a private IPv4 CIDR block. A common and irreversible mistake is choosing a CIDR block that is too small (e.g., /24, which provides only 251 usable IPs). This will severely constrain future growth, making it impossible to add new subnets or resources once the IP space is exhausted.49  
  * **Mitigation:** Plan for growth from the beginning. For most applications, it is best practice to start with a large CIDR block, such as /16 (e.g., 10.0.0.0/16), which provides 65,531 usable IPs. This large address space can then be carved up into smaller subnets for different application tiers and Availability Zones, providing ample room for future expansion.  
* **Over-reliance on the Default VPC:** Every AWS account comes with a default VPC in each region to help users get started quickly. However, these default VPCs are not designed with production security best practices in mind. For instance, their default subnets are public, meaning any EC2 instance launched into them is assigned a public IP address and is exposed to the internet.49  
  * **Mitigation:** For any application beyond simple experimentation, create a **custom VPC**. This gives you full control over the network architecture, including the CIDR block, subnet sizing, route tables, and internet access, allowing you to build a secure and properly segmented network environment.

#### **Security: A Porous Perimeter**

* **Public Subnets for Everything:** A frequent security anti-pattern is placing all resources, including databases and backend application servers, into public subnets. A public subnet is one whose route table has a route to an Internet Gateway. This configuration unnecessarily exposes internal components to the public internet, dramatically increasing the attack surface.49  
  * **Mitigation:** Implement a multi-tier architecture using both public and private subnets.  
    * **Public Subnets:** Should only contain internet-facing resources, such as load balancers, NAT Gateways, or bastion hosts.  
    * **Private Subnets:** Should contain all internal resources, such as application servers, databases, and caching layers. These resources cannot be reached directly from the internet, providing a critical layer of security.  
* **Misconfiguring NACLs and Security Groups:** It is common for engineers to be confused about the difference between Security Groups and Network Access Control Lists (NACLs) or to rely on only one of them.  
  * **Mitigation:** Understand their distinct roles and use them in concert.  
    * **Security Groups:** Act as a stateful firewall at the instance level. They control traffic in and out of an EC2 instance. They should be your primary tool for fine-grained access control (e.g., "allow traffic on port 8080 from the load balancer's security group").  
    * **Network ACLs:** Act as a stateless firewall at the subnet level. They control traffic in and out of a subnet. They are best used as a secondary, broader layer of defense, such as to explicitly block a known malicious IP address at the subnet boundary.49

The cost of data transfer within AWS is a direct, and often surprising, consequence of VPC design. While architects correctly follow the best practice of deploying applications across multiple Availability Zones for high availability, they often overlook the cost implications.52 AWS provides free data transfer within a single AZ, but it charges for every gigabyte of data transferred

*between* AZs.53 A "chatty" application with services communicating frequently across AZs can inadvertently generate thousands of dollars in data transfer fees. This reveals that VPC design is not just about IP addressing and security; it is also about data locality. The placement of NAT Gateways further compounds this issue. If a resource in AZ-A uses a NAT Gateway in AZ-B, it pays a cross-AZ data transfer fee on top of the standard NAT Gateway data processing fee, effectively being charged twice for the same byte of data.54

#### **Diagram: Best-Practice VPC Architecture**

This diagram shows a best-practice VPC design spanning two Availability Zones. The VPC has a large /16 CIDR range. Each AZ contains a public subnet and a private subnet. The public subnets house NAT Gateways and the public-facing nodes of an Application Load Balancer. The private subnets house the application's EC2 instances. An Internet Gateway provides internet access for the public subnets, while the NAT Gateways provide outbound-only internet access for the private subnets. This architecture provides security, scalability, and high availability.

\!(https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)  
Source: 16

### **7.2. Cost Optimization Strategies**

#### **NAT Gateway: The Silent Bill Inflator**

NAT Gateways are a common source of unexpected costs in a VPC. They have both a fixed hourly charge and, more significantly, a per-gigabyte data processing charge for all traffic that passes through them.53 A large portion of this traffic is often destined for other AWS services within the same region.

* **Strategy: Use VPC Endpoints:** This is the most effective way to reduce NAT Gateway costs. VPC Endpoints provide a private connection between your VPC and supported AWS services, allowing traffic to bypass the NAT Gateway.  
  * **Gateway Endpoints:** These are free to use and are available for Amazon S3 and DynamoDB. By creating a Gateway Endpoint and adding a route to it in your private subnet's route table, all traffic to S3 or DynamoDB in that region will be routed through the endpoint instead of the NAT Gateway, completely eliminating the data processing charges for that traffic.49  
  * **Interface Endpoints (AWS PrivateLink):** These are available for most other AWS services. They have an hourly charge and a data processing fee, but these are often lower than the NAT Gateway charges they replace. Crucially, they also eliminate cross-AZ data transfer fees for traffic to the service, which can result in significant net savings.53

#### **Data Transfer Costs**

* **Minimize Cross-AZ Traffic:** As highlighted previously, data transfer between AZs is a key cost driver. Design applications to minimize "chatty" cross-AZ communication. For services that have high inter-communication needs, consider placing them within the same AZ and using other high-availability strategies.  
* **Same-AZ NAT Gateways:** A simple but often overlooked configuration mistake is having resources in one AZ use a NAT Gateway in a different AZ.  
  * **Strategy:** Ensure that each AZ that requires outbound internet access has its own NAT Gateway. Configure the route tables for the private subnets in a given AZ to point to the NAT Gateway in that *same* AZ. This avoids the unnecessary cross-AZ data transfer fee on top of the NAT Gateway processing fee.54

#### **Resource Cleanup**

* **Idle NAT Gateways:** A provisioned NAT Gateway incurs an hourly charge even if no data is passing through it.  
  * **Strategy:** Regularly audit your VPCs to identify and delete any NAT Gateways that are no longer in use, such as those in abandoned development or testing environments.54  
* **Unattached Elastic IPs:** An Elastic IP address that is not associated with a running EC2 instance incurs a small hourly charge.  
  * **Strategy:** Periodically scan your account for unassociated Elastic IPs and release them to avoid these unnecessary charges.

---

## **8\. Conclusion: Cross-Cutting Best Practices**

While each AWS service has its unique pitfalls and optimization strategies, several cross-cutting principles apply universally and are fundamental to building well-architected systems on the cloud.

* **Tag Everything:** A consistent and comprehensive resource tagging strategy is not optional; it is the foundation of effective cloud governance. Tags are key-value pairs that can be attached to any AWS resource. They enable critical functions such as:  
  * **Cost Allocation:** By tagging resources with identifiers like project, team, or cost-center, organizations can accurately track and allocate cloud spend, fostering accountability.  
  * **Automation:** Tags can be used to drive automation. For example, a script can identify all resources with an environment:dev tag and automatically stop them outside of business hours.  
  * **Security and Access Control:** IAM policies can be written to grant permissions based on tags, allowing for fine-grained, attribute-based access control.  
* **IAM is Your Foundation:** The principle of least privilege is the single most effective security control in AWS. Every user, role, and resource should be granted only the minimum permissions necessary to perform its function. Well-crafted IAM policies, regularly reviewed and audited, are the bedrock of a secure cloud environment. Avoid using wildcard (\*) permissions in production policies and leverage tools like IAM Access Analyzer to identify and remediate overly permissive access.  
* **Embrace Infrastructure as Code (IaC):** Manually creating and configuring resources through the AWS Management Console is prone to human error, leads to configuration drift over time, and is not scalable or repeatable. Adopting Infrastructure as Code (IaC) tools like AWS CloudFormation or Terraform is essential for modern cloud development. By defining your entire infrastructure in code, you gain the ability to:  
  * **Version Control:** Track changes to your infrastructure just like you do with your application code.  
  * **Peer Review:** Implement a review process for infrastructure changes, catching potential misconfigurations or security issues before they are deployed.  
  * **Automate Deployments:** Create consistent, repeatable, and automated deployments across all your environments (dev, staging, production), embedding all the security, performance, and cost best practices from this guide directly into your development workflow.4

By internalizing these cross-cutting principles and applying the service-specific guidance detailed in this report, software engineers can move beyond simply writing code to architecting and operating applications that are secure, resilient, performant, and cost-efficient in the AWS cloud.

---

## **9\. References**

* 1 Datacamp. (n.d.).  
  *Top AWS Services for Developers*.  
* 2 Coherence. (n.d.).  
  *AWS Services for Developers: Core Offerings*.  
* 3 Amazon Web Services. (n.d.).  
  *Modern application development on AWS*.  
* 4 Kumar, A. (2023).  
  *Avoiding Pitfalls: 100 Common AWS Mistakes When Building a Project*. Medium.  
* 11 Datadog. (n.d.).  
  *Top 5 AWS EC2 Performance Problems*.  
* 10 Cloud with Usama. (2023).  
  *Understanding the Common Issues in AWS EC2 Instance*. Medium.  
* 8 Middleware. (n.d.).  
  *10 Proven Practices to Reduce AWS Costs*.  
* 17 Spot.io. (n.d.).  
  *12 Great Ways to Save on AWS*.  
* 18 Spacelift. (n.d.).  
  *AWS Cost Optimization: 15+ Ways to Reduce Your AWS Bill*.  
* 5 Cloud Security Alliance. (2024).  
  *AWS S3 Bucket Security: The Top CSPM Practices*.  
* 19 GeeksforGeeks. (2025).  
  *5 Common Mistakes to Avoid When Using AWS S3*.  
* 20 Check Point. (n.d.).  
  *Top 3 S3 Bucket Security Issues*.  
* 22 Wiz. (2025).  
  *S3 cost optimization: A practical guide*.  
* 23 Amazon Web Services. (n.d.).  
  *Optimize costs for Amazon S3*.  
* 21 Finout. (n.d.).  
  *7 Ways to Reduce Your AWS S3 Costs*.  
* 28 PCG. (n.d.).  
  *AWS Lambda: Avoid these common pitfalls*.  
* 27 CSJCode. (2024).  
  *Avoid these AWS Lambda mistakes: Checklist, Symptoms, Solutions*. Medium.  
* 31 Netbird. (n.d.).  
  *AWS Lambda Serverless Security*.  
* 30 Edgedelta. (n.d.).  
  *How to Reduce Amazon Lambda Cost? 10 Methods \+ Plus Bonus Tip*.  
* 29 Sedai. (2025).  
  *Strategies for AWS Lambda Cost Optimization*.  
* 33 The Burning Monk. (2022).  
  *The best ways to save money on Lambda*.  
* 7 Reintech. (2024).  
  *Troubleshooting AWS RDS: Common Issues and How to Fix Them*.  
* 6 Wiz. (n.d.).  
  *Top AWS Security Risks and How to Mitigate Them*.  
* 35 AWS re:Post. (n.d.).  
  *How do I resolve problems connecting to my Amazon RDS DB instance?*.  
* 9 Sedai. (2025).  
  *Cost Optimization Strategies for Amazon RDS in 2025*.  
* 38 Amazon Web Services. (2023).  
  *Optimizing costs in Amazon RDS*.  
* 48 ThinkCloudly. (n.d.).  
  *Advantages and Disadvantages of AWS DynamoDB*.  
* 40 MoldStud. (2025).  
  *Common Pitfalls in DynamoDB and S3 Integration \- How to Avoid them for Seamless Performance*.  
* 56 Amazon Web Services. (n.d.).  
  *Amazon DynamoDB Developer Guide: Cost Optimization*.  
* 46 Amazon Web Services. (n.d.).  
  *Amazon DynamoDB Pricing*.  
* 57 Amazon Web Services. (n.d.).  
  *How AWS Pricing Works: AWS Cost Optimization*.  
* 41 Amazon Web Services. (n.d.).  
  *Amazon DynamoDB Developer Guide: Introduction*.  
* 47 AWS re:Post. (n.d.).  
  *A step-by-step guide to optimizing Amazon DynamoDB costs with CUDOS Dashboard*.  
* 49 Paladin Cloud. (2025).  
  *AWS VPC Security Best Practices*.  
* 50 Hyperglance. (n.d.).  
  *13 AWS VPC Security Best Practices*.  
* 52 Amazon Web Services. (n.d.).  
  *Amazon VPC User Guide: Security best practices*.  
* 51 Datadog. (n.d.).  
  *Best practices for securely configuring Amazon VPC*.  
* 54 nOps. (2025).  
  *How to Reduce NAT Gateway Costs*.  
* 55 AWS re:Post. (n.d.).  
  *How can I reduce data transfer costs for my NAT gateway in Amazon VPC?*.  
* 58 Amazon Web Services. (n.d.).  
  *Amazon VPC User Guide: Pricing for NAT gateways*.  
* 53 CloudKeeper. (n.d.).  
  *AWS Cloud Cost Optimization: NAT Gateway*.  
* 12 Amazon Web Services. (n.d.).  
  *Amazon EC2 Auto Scaling User Guide*.  
* 13 Amazon Web Services. (n.d.).  
  *What is Amazon EC2 Auto Scaling?*.  
* 15 Amazon Web Services. (n.d.).  
  *Tutorial: Set up a scaled and load-balanced application*.  
* 14 Tutorials Dojo. (n.d.).  
  *AWS Auto Scaling*.  
* 24 NetApp. (2024).  
  *Create S3 lifecycle configuration*.  
* 25 AWS re:Post. (n.d.).  
  *How do I use a lifecycle rule to empty my S3 bucket?*.  
* 26 Serverless.com. (n.d.).  
  *What is AWS Lambda?*.  
* 36 Amazon Web Services. (n.d.).  
  *Amazon RDS Read Replicas*.  
* 34 Amazon Web Services. (n.d.).  
  *Getting Started with Amazon RDS: Concepts*.  
* 37 KodeKloud. (n.d.).  
  *AWS RDS Replication Types Introduction*.  
* 42 ScyllaDB. (n.d.).  
  *DynamoDB Hot Partition*.  
* 43 Ramanan, M. (2024).  
  *How to identify hot partition issues with DynamoDB*. Medium.  
* 44 Reddit. (n.d.).  
  *Understanding "hot partitions" in DynamoDB for IoT data*.  
* 45 Amazon Web Services. (n.d.).  
  *Amazon DynamoDB Developer Guide: Designing partition keys to distribute workloads evenly*.  
* 39 Weiskopf, D. (n.d.).  
  *DynamoDB: Know Your Access Patterns*. Medium.  
* 32 Amazon Web Services. (n.d.).  
  *Serverless Multi-Tier Architectures with Amazon API Gateway and AWS Lambda: Sample architecture patterns*.  
* 8 Middleware.io. (2025).  
  *Proven Practices to Reduce AWS Cost*.  
* 19 GeeksforGeeks. (2025).  
  *Common Mistakes to Avoid When Using AWS S3*.  
* 22 Wiz. (2025).  
  *S3 Cost Optimization*.  
* 27 CSJCode. (2024).  
  *Avoid These AWS Lambda Mistakes*.  
* 29 Sedai.io. (2025).  
  *Strategies for AWS Lambda Cost Optimization*.  
* 7 Reintech.io. (2024).  
  *Troubleshooting AWS RDS: Common Issues*.  
* 38 Amazon Web Services. (2023).  
  *Optimizing Costs in Amazon RDS*.  
* 40 MoldStud. (2025).  
  *Common Pitfalls in DynamoDB and S3 Integration*.  
* 16 Amazon Web Services. (n.d.).  
  *VPC with public and private subnets (NAT)*.  
* 49 Paladin Cloud. (2025).  
  *AWS VPC Security Best Practices*.  
* 54 nOps. (2025).  
  *Reduce NAT Gateway Costs*.  
* 36 Amazon Web Services. (n.d.).  
  *Amazon RDS Read Replicas*.  
* 16 Amazon Web Services. (n.d.).  
  *VPC with public and private subnets (NAT)*.

#### **Works cited**

1. Top AWS Services for Developers \- DataCamp, accessed September 15, 2025, [https://www.datacamp.com/blog/aws-services-for-developers](https://www.datacamp.com/blog/aws-services-for-developers)  
2. AWS Services for Developers: Core Offerings \- Coherence, accessed September 15, 2025, [https://www.withcoherence.com/articles/aws-services-for-developers-core-offerings](https://www.withcoherence.com/articles/aws-services-for-developers-core-offerings)  
3. Modern Application Development Services | AWS, accessed September 15, 2025, [https://aws.amazon.com/modern-apps/services/](https://aws.amazon.com/modern-apps/services/)  
4. Avoiding Pitfalls: 100 Common AWS Mistakes When Building a ..., accessed September 15, 2025, [https://medium.com/@ajaykumar426344/avoiding-pitfalls-100-common-aws-mistakes-when-building-a-project-5595306afc31](https://medium.com/@ajaykumar426344/avoiding-pitfalls-100-common-aws-mistakes-when-building-a-project-5595306afc31)  
5. Securing AWS S3 Buckets: Risks and Best Practices | CSA, accessed September 15, 2025, [https://cloudsecurityalliance.org/blog/2024/06/10/aws-s3-bucket-security-the-top-cspm-practices](https://cloudsecurityalliance.org/blog/2024/06/10/aws-s3-bucket-security-the-top-cspm-practices)  
6. 9 All-Too-Common AWS Security Risks \- Wiz, accessed September 15, 2025, [https://www.wiz.io/academy/aws-security-risks](https://www.wiz.io/academy/aws-security-risks)  
7. Troubleshooting Common AWS RDS Issues and Errors | Reintech ..., accessed September 15, 2025, [https://reintech.io/blog/troubleshooting-aws-rds-common-issues](https://reintech.io/blog/troubleshooting-aws-rds-common-issues)  
8. 10 Proven Practices To Reduce AWS Cost by 40% (Tools and Tips), accessed September 15, 2025, [https://middleware.io/blog/proven-practices-to-reduce-aws-cost/](https://middleware.io/blog/proven-practices-to-reduce-aws-cost/)  
9. Cost Optimization Strategies for Amazon RDS in 2025 \- Sedai, accessed September 15, 2025, [https://www.sedai.io/blog/cost-optimization-strategies-for-amazon-rds-in-2025](https://www.sedai.io/blog/cost-optimization-strategies-for-amazon-rds-in-2025)  
10. Understanding the Common Issues in AWS EC2 Instance | by Usama Malik \- Medium, accessed September 15, 2025, [https://medium.com/@cloudwithusama/understanding-the-common-issues-in-aws-ec2-instance-11438f33356a](https://medium.com/@cloudwithusama/understanding-the-common-issues-in-aws-ec2-instance-11438f33356a)  
11. The Top 5 AWS EC2 Performance Problems \- scmGalaxy, accessed September 15, 2025, [https://www.scmgalaxy.com/slides/datadog/slides/Top-5-AWS-EC2-Performance-Problems.pdf](https://www.scmgalaxy.com/slides/datadog/slides/Top-5-AWS-EC2-Performance-Problems.pdf)  
12. Amazon EC2 Auto Scaling \- User Guide, accessed September 15, 2025, [https://docs.aws.amazon.com/pdfs/autoscaling/ec2/userguide/as-dg.pdf](https://docs.aws.amazon.com/pdfs/autoscaling/ec2/userguide/as-dg.pdf)  
13. Amazon EC2 Auto Scaling \- AWS Documentation, accessed September 15, 2025, [https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)  
14. AWS Auto Scaling Cheat Sheet \- Tutorials Dojo, accessed September 15, 2025, [https://tutorialsdojo.com/aws-auto-scaling/](https://tutorialsdojo.com/aws-auto-scaling/)  
15. Tutorial: Set up a scaled and load-balanced application \- Amazon EC2 Auto Scaling, accessed September 15, 2025, [https://docs.aws.amazon.com/autoscaling/ec2/userguide/tutorial-ec2-auto-scaling-load-balancer.html](https://docs.aws.amazon.com/autoscaling/ec2/userguide/tutorial-ec2-auto-scaling-load-balancer.html)  
16. Example: VPC with servers in private subnets and NAT \- Amazon ..., accessed September 15, 2025, [https://docs.aws.amazon.com/vpc/latest/userguide/VPC\_Scenario2.html](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)  
17. AWS Cost Savings: 12 Great Ways to Save on AWS \- Spot.io, accessed September 15, 2025, [https://spot.io/resources/aws-cost-optimization/aws-cost-savings-12-great-ways-to-save-on-aws/](https://spot.io/resources/aws-cost-optimization/aws-cost-savings-12-great-ways-to-save-on-aws/)  
18. AWS Cost Optimization: Strategies, Best Practices, and Tools \- Spacelift, accessed September 15, 2025, [https://spacelift.io/blog/aws-cost-optimization](https://spacelift.io/blog/aws-cost-optimization)  
19. 5 Common Mistakes to Avoid When Using AWS S3 \- GeeksforGeeks, accessed September 15, 2025, [https://www.geeksforgeeks.org/blogs/common-mistakes-to-avoid-when-using-aws-s3/](https://www.geeksforgeeks.org/blogs/common-mistakes-to-avoid-when-using-aws-s3/)  
20. Top 3 S3 Bucket Security Issues \- Check Point Software, accessed September 15, 2025, [https://www.checkpoint.com/cyber-hub/cloud-security/what-is-aws-security/s3-bucket-security/top-3-s3-bucket-security-issues/](https://www.checkpoint.com/cyber-hub/cloud-security/what-is-aws-security/s3-bucket-security/top-3-s3-bucket-security-issues/)  
21. 7 Ways to Reduce Your AWS Cost \- Finout, accessed September 15, 2025, [https://www.finout.io/blog/reduce-aws-s3-cost](https://www.finout.io/blog/reduce-aws-s3-cost)  
22. S3 Cost Optimization: How To Reduce Amazon S3 Storage Spend ..., accessed September 15, 2025, [https://www.wiz.io/academy/s3-cost-optimization](https://www.wiz.io/academy/s3-cost-optimization)  
23. Optimizing storage costs using Amazon S3 \- AWS, accessed September 15, 2025, [https://aws.amazon.com/s3/cost-optimization/](https://aws.amazon.com/s3/cost-optimization/)  
24. Create S3 lifecycle configuration \- NetApp documentation, accessed September 15, 2025, [https://docs.netapp.com/us-en/storagegrid/s3/create-s3-lifecycle-configuration.html](https://docs.netapp.com/us-en/storagegrid/s3/create-s3-lifecycle-configuration.html)  
25. Empty an Amazon S3 bucket with a lifecycle configuration rule \- AWS re:Post, accessed September 15, 2025, [https://repost.aws/knowledge-center/s3-empty-bucket-lifecycle-rule](https://repost.aws/knowledge-center/s3-empty-bucket-lifecycle-rule)  
26. AWS Lambda: The Ultimate Guide \- Serverless, accessed September 15, 2025, [https://www.serverless.com/aws-lambda](https://www.serverless.com/aws-lambda)  
27. AVOID these AWS Lambda MISTAKES (checklist \+ symptoms ..., accessed September 15, 2025, [https://medium.com/@csjcode/avoid-these-aws-lambda-mistakes-checklist-symptoms-solutions-e32124cb2118](https://medium.com/@csjcode/avoid-these-aws-lambda-mistakes-checklist-symptoms-solutions-e32124cb2118)  
28. AWS Lambda: Avoid these common pitfalls \- PCG.io, accessed September 15, 2025, [https://pcg.io/insights/aws-lambda-avoid-these-common-pitfalls/](https://pcg.io/insights/aws-lambda-avoid-these-common-pitfalls/)  
29. Strategies for AWS Lambda Cost Optimization \- Sedai, accessed September 15, 2025, [https://www.sedai.io/blog/strategies-for-aws-lambda-cost-optimization](https://www.sedai.io/blog/strategies-for-aws-lambda-cost-optimization)  
30. Reduce Amazon Lambda Cost: 10 Effective Methods \- Edge Delta, accessed September 15, 2025, [https://edgedelta.com/company/blog/how-to-reduce-amazon-lamba-cost](https://edgedelta.com/company/blog/how-to-reduce-amazon-lamba-cost)  
31. AWS Lambda Serverless Security. Mistakes, Oversights, and Potential Vulnerabilities, accessed September 15, 2025, [https://netbird.io/knowledge-hub/aws-lambda-serverless-security](https://netbird.io/knowledge-hub/aws-lambda-serverless-security)  
32. Sample architecture patterns \- AWS Serverless Multi-Tier Architectures with Amazon API Gateway and AWS Lambda, accessed September 15, 2025, [https://docs.aws.amazon.com/whitepapers/latest/serverless-multi-tier-architectures-api-gateway-lambda/sample-architecture-patterns.html](https://docs.aws.amazon.com/whitepapers/latest/serverless-multi-tier-architectures-api-gateway-lambda/sample-architecture-patterns.html)  
33. The best ways to save money on Lambda | theburningmonk.com, accessed September 15, 2025, [https://theburningmonk.com/2022/07/the-best-ways-to-save-money-on-lambda/](https://theburningmonk.com/2022/07/the-best-ways-to-save-money-on-lambda/)  
34. Key concepts and architecture of Amazon RDS \- Amazon Relational Database Service, accessed September 15, 2025, [https://docs.aws.amazon.com/AmazonRDS/latest/gettingstartedguide/concepts.html](https://docs.aws.amazon.com/AmazonRDS/latest/gettingstartedguide/concepts.html)  
35. Troubleshoot Amazon RDS DB instance connection issues \- AWS re:Post, accessed September 15, 2025, [https://repost.aws/knowledge-center/rds-cannot-connect](https://repost.aws/knowledge-center/rds-cannot-connect)  
36. Amazon RDS Read Replicas | Cloud Relational Database \- AWS, accessed September 15, 2025, [https://aws.amazon.com/rds/features/read-replicas/](https://aws.amazon.com/rds/features/read-replicas/)  
37. AWS RDS Replication Types Introduction \- KodeKloud Notes, accessed September 15, 2025, [https://notes.kodekloud.com/docs/AWS-Certified-SysOps-Administrator-Associate/Domain-2-Reliability-and-BCP/AWS-RDS-Replication-Types-Introduction](https://notes.kodekloud.com/docs/AWS-Certified-SysOps-Administrator-Associate/Domain-2-Reliability-and-BCP/AWS-RDS-Replication-Types-Introduction)  
38. Optimizing costs in Amazon RDS | AWS Database Blog, accessed September 15, 2025, [https://aws.amazon.com/blogs/database/optimizing-costs-in-amazon-rds/](https://aws.amazon.com/blogs/database/optimizing-costs-in-amazon-rds/)  
39. DynamoDB — Know Your Access Patterns | by Daniel Weiskopf \- Medium, accessed September 15, 2025, [https://medium.com/@danielaaronw/dynamodb-know-your-access-patterns-519c9bf528af](https://medium.com/@danielaaronw/dynamodb-know-your-access-patterns-519c9bf528af)  
40. Avoiding Common Pitfalls in DynamoDB and S3 Integration | MoldStud, accessed September 15, 2025, [https://moldstud.com/articles/p-common-pitfalls-in-dynamodb-and-s3-integration-how-to-avoid-them-for-seamless-performance](https://moldstud.com/articles/p-common-pitfalls-in-dynamodb-and-s3-integration-how-to-avoid-them-for-seamless-performance)  
41. What is Amazon DynamoDB? \- Amazon DynamoDB \- AWS Documentation, accessed September 15, 2025, [https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)  
42. What is a DynamoDB Hot Partition? Definition & FAQs | ScyllaDB, accessed September 15, 2025, [https://www.scylladb.com/glossary/dynamodb-hot-partition/](https://www.scylladb.com/glossary/dynamodb-hot-partition/)  
43. How to identify hot partition issues with DynamoDB ? | by MR \- Medium, accessed September 15, 2025, [https://medium.com/@manojramanann/how-to-identify-hot-partition-issues-with-dynamodb-741abc6c20c3](https://medium.com/@manojramanann/how-to-identify-hot-partition-issues-with-dynamodb-741abc6c20c3)  
44. Understanding Hot Partitions in DynamoDB for IoT Data Storage : r/aws \- Reddit, accessed September 15, 2025, [https://www.reddit.com/r/aws/comments/1jjjwvo/understanding\_hot\_partitions\_in\_dynamodb\_for\_iot/](https://www.reddit.com/r/aws/comments/1jjjwvo/understanding_hot_partitions_in_dynamodb_for_iot/)  
45. Best practices for designing and using partition keys effectively in ..., accessed September 15, 2025, [https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)  
46. Amazon DynamoDB Pricing | NoSQL Key-Value Database, accessed September 15, 2025, [https://aws.amazon.com/dynamodb/pricing/](https://aws.amazon.com/dynamodb/pricing/)  
47. A Step-by-Step Guide to Optimizing Amazon DynamoDB Costs with CUDOS Dashboard, accessed September 15, 2025, [https://repost.aws/articles/ARexp\_3RxiQKOGwXbPaPWXPQ/a-step-by-step-guide-to-optimizing-amazon-dynamodb-costs-with-cudos-dashboard](https://repost.aws/articles/ARexp_3RxiQKOGwXbPaPWXPQ/a-step-by-step-guide-to-optimizing-amazon-dynamodb-costs-with-cudos-dashboard)  
48. 10 Advantages & Disadvantages of AWS DynamoDB \- ThinkCloudly, accessed September 15, 2025, [https://thinkcloudly.com/blog/aws/advantages-disadvantages-aws-dynamodb/](https://thinkcloudly.com/blog/aws/advantages-disadvantages-aws-dynamodb/)  
49. AWS VPC: Security Best Practices \- Paladin Cloud, accessed September 15, 2025, [https://paladincloud.io/aws-security-risks/aws-vpc-security-best-practices/](https://paladincloud.io/aws-security-risks/aws-vpc-security-best-practices/)  
50. AWS VPC Security: 13 Best Practices \[Updated Guide\] \- Hyperglance, accessed September 15, 2025, [https://www.hyperglance.com/blog/aws-vpc-security-best-practices/](https://www.hyperglance.com/blog/aws-vpc-security-best-practices/)  
51. Best practices for securely configuring Amazon VPC \- Datadog, accessed September 15, 2025, [https://www.datadoghq.com/blog/vpc-components/](https://www.datadoghq.com/blog/vpc-components/)  
52. Security best practices for your VPC \- Amazon Virtual Private Cloud \- AWS Documentation, accessed September 15, 2025, [https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)  
53. AWS Cloud Cost Optimization in NAT Gateway | CloudKeeper, accessed September 15, 2025, [https://www.cloudkeeper.com/insights/blog/aws-cloud-cost-optimization-nat-gateway](https://www.cloudkeeper.com/insights/blog/aws-cloud-cost-optimization-nat-gateway)  
54. AWS NAT Gateway Costs and Ways to Reduce Them: A Guide \- nOps, accessed September 15, 2025, [https://www.nops.io/blog/reduce-nat-gateway-costs-using-nops-deep-insight-service/](https://www.nops.io/blog/reduce-nat-gateway-costs-using-nops-deep-insight-service/)  
55. How do I reduce data transfer charges for my NAT gateway in Amazon VPC? \- AWS re:Post, accessed September 15, 2025, [https://repost.aws/knowledge-center/vpc-reduce-nat-gateway-transfer-costs](https://repost.aws/knowledge-center/vpc-reduce-nat-gateway-transfer-costs)  
56. Optimizing costs on DynamoDB tables \- Amazon DynamoDB, accessed September 15, 2025, [https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-cost-optimization.html](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-cost-optimization.html)  
57. AWS Cost Optimization \- How AWS Pricing Works \- AWS Documentation, accessed September 15, 2025, [https://docs.aws.amazon.com/whitepapers/latest/how-aws-pricing-works/aws-cost-optimization.html](https://docs.aws.amazon.com/whitepapers/latest/how-aws-pricing-works/aws-cost-optimization.html)  
58. Pricing for NAT gateways \- Amazon Virtual Private Cloud, accessed September 15, 2025, [https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-pricing.html](https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-pricing.html)
