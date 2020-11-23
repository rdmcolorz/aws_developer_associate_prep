# AWS X-Ray

[Lesson link](https://youtu.be/RrKRN9zRBWs?t=10522)
Help developers analyze and debug applications utilizing micro-service architecture. It is a distributed tracing / performance monitoring system.

- **Micro-services**:
  - Micro-services is an architectural and organizational approach to software development where software is composed of small independent services that communicate over well-defined APIs.
  - Micro-service architectures make applications easier to scale and faster to develop, enabling innovation and accelerating time-to-market for new features.
  - Ex: Serverless storage, container tasks, databases, notifications, queueing, streaming, serverless functions.
- **Distributed Tracing**
  - Also called distributed request tracing, is a method used to profile and monitor apps, especially those built using a micro-service architecture. Distributing tracing helps pinpoint failures occur and what causes poor performance.
- **Performance Monitoring**
  - Monitoring and management of performance and availability of software apps. APM strives to detect and diagnose complex application performance problems to maintain an expected level of service.
- **Similar third-part service to X-ray**
  - Datadog, New Relic, SignalFx, lumigo
- **X-Ray is a Distributed Tracing system**
  - Collects data about requests that your application serves
  - View, filter collected data to identify issues and avenues for optimization.
  - For any traced request to your application, you can see detailed information not only about the request and response, but also about calls that your application makes to downstream AWS resources, micro-services, databases and HTTP web APIs.

## The anatomy of X-Ray

|              |       |                  |       |               |
| :----------: | :---: | :--------------: | :---: | :-----------: |
| X-Ray Daemon |   ➡️   |    X-Ray API     |   ➡️   | X-Ray Console |
|      ⬆️       |  ↘️↖️   |        ↕️         |       |               |
|  X-Ray SDK   |       | AWS SDK/ AWS CLI |       |               |

- The X-Ray SDK provides:
  - Interceptors to add to your code to trace incoming HTTP requests
  - Client handlers to instrument AWS SDK clients that your application uses to call other AWS services
  - An HTTP Client to instrument calls to other internal and external HTTP web services.

## Instrumentation

- Instrumenting is the ability to monitor or measure the level of a product's performance, to diagnose errors, and to write trace information.
- Ex:
  
  ``` javascript
    var app  = express();
    var AWSXRay = require("aws-xray-sdk")

    // Start tracing 
    app.use(AWSXRay.express.openSegment("MyApp"));

    app.get("/", function(req, res) {
        res.render("index");
    });

    # End tracing
    app.use(AWSXRay.express.closeSegment());
  ```

## X-Ray Daemon

- Instead of sending trace data directly to X-Ray, the SDK sends JSON segment documents to a daemon process listening to UDP traffic on port 2000.
- The X-Ray Daemon buffers segments in a queue and uploads them to X-Ray in batches.
- The daemon is available for Linux, Windows, Mac, and included on EB and Lambda platforms.
- X-Ray uses trace data from the AWS resources that power your cloud applications to generate a detailed service graph.

## X-Ray Concepts

- X-Ray receives data from services as segments.
- X-Ray groups segments that have a common request into traces.
- X-Ray processes the traces to generate a service graph that provides a visual representation of your app.
  - **Segments**
    - The compute resources running your application logic sends data about their work as segments.
    - A segment can send the following information:
      - **The host**: hostname, alias or IP address
      - **The request**: method, client, address, path, user agent
      - **The response**: status, content
      - **The work done**: start and end times, subsegments
      - **Issues that occur**: errors, faults and exceptions, including automatic capture of exception stacks.
  - **Subsegments**
    - Subsegments provide more granular timing information and details about downstream calls that your app made to fulfill the original request.
    - A subsegment can contain additional details about a call to an AWS service, an external HTTP API, or an SQL database.
    - ![subsegments](./subsegments.png)
    - Define arbitrary subsegments to instrument specific functions or lines of code in your app.
    - Ex:
  
  ``` javascript
    const AWS = require("aws-sdk")
    const xray = require("aws-xray-sdk-core")

    exports.handler = (event, context, callback) => {
        const segment = xray.getSegment()

        const subsegment = segment.addNewSubsegment("Mining")
        mining()
        subsegment.close()

        callback();
    }

    function mining() {
        console.log("mining for precious minerals")
    }
  ```

  - **Service Graph**
    - Shows client, front-end services, and back-end services
    - Use the service graph to improve performance by identifying bottlenecks, latency spikes, and etc.
    ![scorekeep-servicemap](./scorekeep-servicemap.png)
  - **Traces**
    - A trace collects all segments generated by a single request.
    - A trace ID tracks the path of a request through your application.
    - The first supported service that the HTTP request interacts with adds a trace ID header to the request, and propagates it downstream to track the latency, disposition, and other request data.
  - **Sampling**
    - The X-Ray SDK uses a sampling algrithm to determine which requests get traced.
    - By default, the X-Ray SDK records the first request each second, and 5% of any additional requests.
    - Sampling helps you save money by reducing the amount of traces for high-volume and unimportant requests.
  - **Tracing header**
    - All requests are traced, up to a configurable minimum, and after reaching that minimum, a percentage of requests are traced to avoid unnecessary cost.
    - The sampling decision and trace ID are added to HTTP requests in tracing headers named `X-Amzn-Trace-Id`.
  - **Filter expressions**
    - Even with sampling, a complex application generates a lot of data, so we use filter expressions to narrow down specific path or users.
  - **Groups**
    - You can assign a Filter expression to a Group so you can easily view it.
    - You can call the group by name or by Amazon Resource Name (ARN) to generate its own service graph, trace summaries, and Amazon CloudWatch metrics.
    - Updating a group's filter expression doesn't change data that's already recorded. The update applies only to subsequent traces, which can result in a merged graph of old and new expressions.
    - To avoid merging data, you need to delete the current group and create a new one.
  - **Annotations and Metadata**
    - You can add other information to the segment document as annotations and metadata.
    - Annotations and metadata are aggregated at the trace level, and can be added to any segment/ subsegment.
    - **Annotations**:
      - key-value pairs that are indexed for use with filter expressions. X-Ray indexes up to 50 annotations per trace.
      - Use to record data you want to use to group traces in the console, or when calling the `GetTraceSummaries` API.
    - **Metatdata**:
      - key-value pairs that are not indexed. The values can be of any type, including objects and lists.
      - Use to record data you want to store in the trace but don't need to use for searching traces.
  - **Errors, Faults, and Exceptions**
    - When a exception occurs while your app is serving an instrumented request, the X-Ray SDK records exception details and the stack trace (if available).
    - Errors are 400/ 4xx errors
    - Faults are 400/ 5xx errors
    - Throttle is 429 Too Many Requests

## AWS Service Integrations

- Lambda, ELB, API Gateway, SNS, SQS, App Mesh, CloudTrail, EC2, CloudWatch, ECS, AWS Config, ECS Fargate, EB.

## Supported Languages

- Go, NodejS, Ruby, Java, PHP, Python, ASP.NET
