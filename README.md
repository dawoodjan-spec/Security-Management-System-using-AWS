<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Security Monitoring System

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-security-monitoring)

**Author:** Muhammad Dawood Jna  
**Email:** muhammaddawoodjan989@gmail.com

---

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_reghtjy)

---

## Introducing Today's Project!

In this project, I will demonstrate how to securely store, manage and monitor sensitive information (such as API keys and passwords) using AWS. I'm doing this project to learn how to store secrets safely, track who accesses them, and receive real-time alerts when unusual access attempts occur.
I'll use AWS services like Secrets Manager, CloudTrail, CloudWatch, and SNS to create a secure and automated system that not only protects our data but also keeps us informed about potential security issues.

### Tools and concepts

Services I used were AWS Secrets Manager, CloudTrail, CloudWatch, and SNS. 

Key concepts I learnt include securely storing sensitive data as secrets, tracking access through CloudTrail logs, creating CloudWatch alarms based on specific metric filters, and sending real-time alerts using SNS.

This project helped me understand how to build a basic security monitoring system in AWS using event-driven architecture.

### Project reflection

This project took me almost 2 hours. The most challenging part was troubleshooting the CloudWatch alarm and figuring out why it wasn’t triggering as expected. Reconfiguring the settings and testing different combinations was time-consuming. But the most rewarding moment was finally receiving the email notification after retrieving the secret, it confirmed that everything was working correctly.

---

## Create a Secret

Secrets Manager is a service of AWS through which we can protect and save our sensitive information like password, API keys, credentials or other sensitive data securely.

To set up for my project, I created a secret called gmailPass that contains the password of my gmail account.

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_o5p6q7r8)

---

## Set Up CloudTrail

AWS CloudTrail is a monitoring service, like an activity recorder throughout our AWS account. It documents every action taken, like who did what, when they did it, and where they did it from. This continuous recording is super valuable for security (spotting unusual activity), troubleshooting (figuring out what changed when something breaks), and meeting compliance requirements. In my project, I am using it to keep an eye on who's accessing our secret.

CloudTrail captures different types of events, each showing you a different view into what's happening in our AWS account:

Management events: These shows our admin actions that configure our AWS resources like creating an EC2 instance, updating a security group, or in our case, accessing a secret. 

Data events: These track high-volume actions that operate ON our resources rather than creating or configuring them  like uploading a file to S3 or running a Lambda function.

Insights events: These detect unusual patterns in our management events, like someone suddenly creating 100x more IAM users than normal.

Network activity events: These track network-related activities, like changes to our VPC configuration or traffic to a subnet.

Different types of API activities include:
Read, Write,  List, Permissions, Tagging, Policy, Login/Auth

### Read vs Write Activity

Read API activity happens when someone views but doesn't change anything. For example, listing our S3 buckets, describing our EC2 instances, or like in this project, viewing (but not changing) metadata about a secret.
Write API activity occurs when changes happen like creating, deleting, modifying resources, or even retrieving the value of a secret (which is what we want to monitor).

---

## Verifying CloudTrail

I retrieved the secret in two ways: First through the Graphical User Interface  and second through the Cloud Shell / Command Line Interface (CLI)

To analyze my CloudTrail events, I visited the Event history, where I found logs of all events in my AWS account. I found that CloudTrail records every API call made. This tells me that CloudTrail helps in tracking user activity and service usage. Then, I filtered the records specifically for AWS Secrets Manager to focus on its related actions.

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_s8t9u0v1)

---

## CloudWatch Metrics

Amazon CloudWatch Logs is a service that helps us bring together our logs from different AWS services, including CloudTrail, for visibility, troubleshooting, and analysis.
It's especially powerful because once our logs are in CloudWatch, we can create alerts based on specific patterns (such as someone accessing our secret), visualize trends, or trigger automated responses.
For this project, I'm sending CloudTrail logs to CloudWatch Logs so I can set up alerts when someone accesses my secret.

CloudTrail's Event History is useful for tracking what actions were taken in our AWS account like who created a resource, deleted a user, or accessed a secret while CloudWatch Logs are better for digging into the details of how things are running inside our applications and services, like errors, performance metrics, or custom logs from Lambda functions.
In simple terms:
CloudTrail = "Who did what and when" (auditing and security)
CloudWatch = "What’s happening inside" (monitoring and troubleshooting)
Both are powerful, but they serve different purposes in keeping our system secure and healthy.

A CloudWatch metwric scans through our logs looking for specific patterns. Instead of us manually reading thousands of log entries, a metric filter automatically detects these patterns and keeps count for us. When setting up a metric, the metric value represents the value that gets recorded when our filter spots a match in the logs. Default value is used when our filter doesn't finds a match in the logs during given time period.

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_a9b0c1d2)

---

## CloudWatch Alarm

A CloudWatch alarm is a monitoring tool that watches a specific metric over time and takes action when the metric crosses a defined threshold. I set my CloudWatch alarm threshold to a specific value such as "greater than or equal to 1 unauthorized Secrets Manager access attempt within 5 minutes", so the alarm will trigger when suspicious activity is detected.
This setup ensures that I get notified immediately through Amazon SNS if someone tries to access secrets without proper permission, helping me respond quickly to potential security threats.

I created an SNS topic along the way. An SNS topic is like a communication channel used to send messages to multiple subscribers at once, such as email addresses, Lambda functions, or other services. My SNS topic is set up to deliver notifications when a CloudWatch alarm is triggered, so I can get immediate alerts (e.g., via email) whenever suspicious activity like unauthorized access to secrets is detected. This allows for quick awareness and response to potential security incidents.


AWS requires email confirmation because it ensures that the recipient actually consents to receiving notifications from an SNS topic. This helps prevent unauthorized subscriptions, spam, or accidental delivery of messages to unintended email addresses

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_fsdghstt)

---

## Troubleshooting Notification Errors

To test my monitoring setup, I retrieved the value of my secret manually, but I didn’t receive any email notification as expected.

When troubleshooting the notification issues, I thought that there could be 5 reasons because of why my monitoring system could be breaking:
1-CloudTrail didn't record the GetSecretValue event.
2-CloudTrail isn't sending logs to CloudWatch.
3-CloudWatch's metric filter isn't filtering logs correctly.
4-CloudWatch's Alarm isn't triggering an action.
5- SNS isn't delivering emails to me.

I initially didn't receive an email before because the CloudWatch alarm wasn't triggering due to the wrong statistic setting. The key solution was to change the Statistic from "Average" to "Sum" and reduce the Period from 5 minutes to 1 minute. This made the alarm more responsive and allowed it to trigger immediately when the secret was accessed.

---

## Success!

To validate that my monitoring system works, I retrieved the secret from AWS Secrets Manager to simulate an access event. I then checked the CloudWatch dashboard and confirmed that my alarm status changed to "In Alarm." I received an email notification as well, which confirmed that the entire setup (from triggering the alarm to receiving the alert) was functioning correctly.

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_ageraergearge)

---

## Comparing CloudWatch with CloudTrail Notifications

In a project extension, I configured direct SNS notifications from CloudTrail because I wanted to compare notification methods of CloudWatch and CloudTrail.



After enabling CloudTrail SNS notifications, my inbox became flooded with emails, every single event was triggering a notification, even routine or harmless ones.
In terms of the usefulness of these emails, I thought it was overwhelming and hard to spot actual security issues among all the noise. It made me realize that it's better to fine-tune what events trigger alerts for example, only sending notifications for unauthorized access attempts or specific API calls so the alerts stay meaningful and actionable.

![Image](http://learn.nextwork.org/blissful_turquoise_brave_goblin/uploads/aws-security-monitoring_d7e8f9g0)

---

---
