### **Basic Concepts & Fundamentals**

These questions test your understanding of what Lambda is and its core components.

**Q1: What is AWS Lambda, and what is its primary use case?**
**Answer:** 
AWS Lambda is a serverless, event-driven compute service that lets you run code without provisioning or managing servers. Its primary use case is to run backend code in response to events like HTTP requests (via API Gateway), file uploads to S3, updates in a DynamoDB table, or messages from a queue. You pay only for the compute time you consume.

**Q2: Explain the basic components of a Lambda function.**
**Answer:**
1.  **Code:** The actual code (in a supported runtime like Python, Node.js, Java, etc.) that you want to execute.
2.  **Configuration:** Settings like memory allocation (128 MB to 10 GB), timeout (up to 15 minutes), IAM execution role, and environment variables.
3.  **Trigger:** The AWS service or resource that invokes the function (e.g., API Gateway, S3, EventBridge).
4.  **Execution Role:** An IAM role that grants the function permission to access other AWS services.

**Q3: What are the main advantages of using AWS Lambda?**
**Answer:**
*   **No Server Management:** AWS fully manages the infrastructure.
*   **Automatic Scaling:** Lambda scales your application by running code in response to each trigger.
*   **Cost-Effective:** You pay only for the number of requests and the compute time (in GB-seconds). There is no charge when your code is not running.
*   **Event-Driven:** Naturally integrates with many AWS services, making it ideal for microservices and event-driven architectures.

**Q4: What is a Lambda "Cold Start"?**
**Answer:** A cold start is the latency incurred when Lambda has to initialize a new execution environment for your function. This involves:
1.  Downloading your code.
2.  Creating the container.
3.  Initializing the runtime and your function code (the `init` phase).
A "warm start" is much faster, as it reuses an existing environment and only runs the handler function.

---

### **Intermediate Configuration & Management**

These questions dive deeper into how Lambda works and how to configure it effectively.

**Q5: How can you mitigate the impact of Cold Starts?**
**Answer:**
*   **Provisioned Concurrency:** This is the most effective solution. It pre-initializes a specified number of execution environments, keeping them "warm" and ready to respond immediately.
*   **Optimize Deployment Package:** Reduce the size of your function's ZIP/JAR by removing unnecessary dependencies. This speeds up the download phase.
*   **Use Simple Runtimes:** Runtimes like Node.js and Python generally have faster initialization times than Java or .NET.
*   **Keep Functions "Warm":** For non-critical functions, you can use a scheduled CloudWatch Event to ping your function every ~5-15 minutes to keep an environment alive.

**Q6: What are Lambda Layers, and what problem do they solve?**
**Answer:** Lambda Layers are a distribution mechanism for libraries, custom runtimes, or other function dependencies. They solve the problem of having large, redundant deployment packages.
*   **Separation of Concerns:** You can separate your custom business logic from shared dependencies.
*   **Faster Deployments:** Since layers are cached and can be reused, your function deployment package is smaller and deploys faster.
*   **Sharing & Reusability:** A single layer can be used by multiple functions across different accounts (if shared appropriately).

**Q7: Explain the different memory settings and their impact.**
**Answer:** You can allocate memory between 128 MB and 10,240 MB. This setting is crucial because:
*   **CPU Power:** CPU power allocated to a function is proportional to the memory configured. More memory = more CPU.
*   **Cost:** Cost is calculated as `(Memory Allocated * Execution Time)`. So, a function that runs faster with more memory might be cheaper than a slower function with less memory.
*   **Optimization:** It's often beneficial to run performance tests with different memory settings to find the most cost-effective and performant configuration.

**Q8: What is the difference between an asynchronous and a synchronous Lambda invocation?**
**Answer:**
*   **Synchronous:** The caller waits for the function to process the event and return a response. Examples: API Gateway, Application Load Balancer, Cognito.
*   **Asynchronous:** The caller places the event in an internal queue and immediately gets an acknowledgment. Lambda then processes the event in a separate process. Examples: S3, SNS, CloudWatch Events/EventBridge. For async invocations, Lambda retries twice (by default) on failure.

---

### **Advanced & Scenario-Based Questions**

These questions test your ability to apply Lambda concepts to real-world architectural problems.

**Scenario 1: High-Volume Image Processing**
**Q: You are designing a system where users upload images to an S3 bucket. Each uploaded image must be resized into three different thumbnails. How would you architect this using Lambda, and what are the key considerations?**

**Answer:**
1.  **Architecture:**
    *   The user uploads an image to the source S3 bucket (`source-images`).
    *   This upload triggers a Lambda function.
    *   The Lambda function (using a library like Pillow for Python or Sharp for Node.js) processes the image and creates three thumbnails.
    *   The function then saves these thumbnails to a different S3 bucket (`thumbnails`).
2.  **Key Considerations:**
    *   **Concurrency & Scaling:** S3 can generate a huge number of events. Lambda will scale automatically, but you must be aware of your account's concurrency limits and request increases if needed.
    *   **Error Handling:** Use an SQS Dead-Letter Queue (DLQ) or a Destination for the async invocation to capture any failed processing events (e.g., corrupted images).
    *   **Performance:** Allocate sufficient memory/CPU to the function, as image processing is computationally intensive. Test to find the optimal memory setting.
    *   **Idempotency
