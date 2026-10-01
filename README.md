# Ansible-Flask-App
## Solution
### OBJECTIVE
Deployed a Flask application on AWS EC2 using Ansible with:

**Manual AWS Configuration**
* 2 Quality/Application EC2 servers
* 1 dedicated MySQL DB EC2 server
* AWS Security Groups
* Application Load Balancer (ALB)
* Target Group on port **5000**
* HTTP listener on port **80**
* ALB health checks
* CloudWatch Dashboard with 4 ALB metrics
* CloudWatch Alarm for **`UnHealthyHostCount > 1`**
* SNS Topic and Email Subscription for alerts

**Ansible Automation**
* Separate Ansible controller
* AWS dynamic inventory using **plugin, filters, and keyed groups**
* Ansible roles and group-specific variables
* GitHub-based application deployment
* Private Flask-to-MySQL communication
* Automated Flask application startup


### ANSIBLE PROJECT STRUCTURE
The Ansible project was organized using dynamic inventory, group variables and roles.
ansible/
├── aws_ec2.yaml
├── playbook.yaml
├── group_vars/
│   ├── Server_Quality.yml
│   ├── Server_Development.yml
│   └── Server_DBSERVER.yml
│
└── roles/
    ├── app/
    │   └── tasks/
    │       ├── main.yml
    │       ├── setup.yml
    │       ├── git.yml
    │       └── deploy.yml
    │
    └── db/
        └── tasks/
            └── main.yml


### AWS SECURITY GROUPS

**Quality/Application Servers**

* TCP 22 — SSH
* TCP 5000 — Flask application with source - Security group name SG-ALB (app load balancer SG which allows internet traffic)

Flask application is accessible using:
`http://<app-load-balancer-Public-IP>`

**DB Server**

* TCP 3306 — MySQL
* Source restricted to `Flask_SG`

This allows MySQL communication only from the Flask application servers, without exposing port 3306 directly to the public internet.

### LOAD BALANCING

**QA Servers – Security Groups & ALB**
**Security Groups**

* Configured a dedicated **ALB Security Group** allowing HTTP **port 80** from `0.0.0.0/0`.
* Attached the ALB Security Group to the **ALB**.
* Allowed **port 5000** on QA EC2 instances with the **ALB Security Group as the source**.
* Restricted direct Internet access to the QA application instances.
* This ensures application traffic reaches EC2 instances only through the ALB.

**Load Balancer**
* Created an **Application Load Balancer (ALB)**.
* Configured an **HTTP listener on port 80**.
* Created a **Target Group** containing QA EC2 instances on **port 5000**.
* Configured health checks:

  * Timeout: **5 seconds**
  * Interval: **30 seconds**
  * Unhealthy threshold: **2**
  * Healthy threshold: **4**

**Traffic Flow**
`Internet → ALB:80 → Target Group:5000 → QA EC2 Instances`

### CLOUDWATCH MONITORING & SNS ALERT

**CloudWatch Dashboard**
  * Created a CloudWatch Dashboard for monitoring the Application Load Balancer.
  * Added 4 ALB metrics:
    RequestCount – total requests received by the ALB.
    TargetResponseTime – response time of the targets.
    HTTPCode_Target_4XX_Count – number of 4XX responses from targets.
    UnHealthyHostCount – number of unhealthy targets.
  * Used the dashboard to monitor ALB and target health.

**CloudWatch Alarm**
  * Created an alarm using UnHealthyHostCount.
  * Condition: UnHealthyHostCount > 1.
  * When the condition is met, the alarm changes to ALARM state.
  * Configured the alarm to trigger an SNS notification.
    
**SNS Configuration**
  * Created an SNS Topic for CloudWatch alerts.
  * Created an Email Subscription under the topic.
  * Entered the email address for receiving alerts.
  * Confirmed the subscription using the confirmation email.
  * Selected the SNS topic as the Alarm action in CloudWatch.

Flow:
UnHealthyHostCount > 1 → CloudWatch Alarm → SNS Topic → Email Subscription → Email Alert


### FINAL DEPLOYMENT FLOW

1. **AWS EC2 infrastructure**
   ↓
2. **Configure AWS Security Groups**
   ↓
3. **Verify SSH access**
   ↓
4. **Configure Ansible project**
   ↓
5. **Configure AWS dynamic inventory**
   ↓
6. **Create `Server_Quality` / `Server_DBSERVER` groups**
   ↓
7. **Configure `group_vars`**
   ↓
8. **Configure Ansible roles**
   ↓
9. **Install MySQL on DB server**
   ↓
10. **Create MySQL application user**
    ↓
11. **Allow Quality → DB :3306**
    ↓
12. **Install Python/Flask dependencies**
    ↓
13. **Retrieve DB private IP dynamically**
    ↓
14. **Fetch Flask code from GitHub**
    ↓
15. **Start Flask on Quality servers :5000**
    ↓
16. **Create Application Load Balancer**
    ↓
17. **Create Target Group with Quality EC2 instances on port 5000**
    ↓
18. **Configure HTTP :80 listener → Target Group :5000**
    ↓
19. **Configure ALB health checks**
    ↓
20. **Verify target health and ALB connectivity**
    ↓
21. **Users access Flask application through ALB :80**
    ↓
22. **ALB forwards requests to healthy Quality Servers :5000**
    ↓
23. **Flask connects to MySQL using DB private IP when required**
    ↓
24. **Response is returned to the user through the ALB**
    ↓
25. **Configure CloudWatch alarm for `UnHealthyHostCount > 1`**
    ↓
26. **Configure SNS topic and email subscription**
    ↓
27. **SNS sends email notification when CloudWatch alarm is triggered**


