[Lesson link]("https://youtu.be/RrKRN9zRBWs?t=8234")
## AWS Elastic Container Service (ECS)
Fully-managed container orchestration service. Highly secure, reliable, and scalable way to run containers.\

## Notes
Components of ECS
- **Cluster**
Multiple EC2 (Elastic Computing Cloud) instances which will house the docker containers.
- **Task Definition**
A JSON file that defines the configuration of (up to 10) containers you want to run.
- **Task**
Launches containers defined in Task definition.\
Tasks do not remain running  once workload is complete.
- **Service**
Ensures tasks remain running
- **Container Agent**
Binary on each EC2 instance which monitors, start and stops tasks.

### Creating cluster
- use spot or demand (spot for saving money)
    - spot instances are unused EC2 instance that is available for less thean the on-demand prices, but can be interrupted. Spot instances are suitable for data analysis, batch jobs, background processing, and optional tasks.
- EC2 instance type
- Number of instances
- EBS Storage volume
- EC2 can be Amazon Linux 1 or Amazon Linux 2
- Choose a VPC or create new VPC (Virtual Private Cloud)
- Assign IAM role
- Option to turn on CloudWatch container insights
- Choose SSH key pair (Not recommended)