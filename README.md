# Prerequisites
#
#
#
- JDK 17 or 21
- Maven 3.9
- MySQL 8

# Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- Rabbitmq
- ElasticSearch
# Database
Here,we used Mysql DB 
sql dump file:
- /src/main/resources/db_backup.sql
- db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < db_backup.sql


# Pipeline Configuration
Plugins:        Git Integration, Maven Integration, Nexus Artifact Uploader, SonarQube Scanner, Build Timestamp, Slack Notification
Tools:          JDK 17, Maven 3.9, SonarQube Scanner 4.7.0.2747
Credentials:    sonartoken, slacktoken, gitlogin, nexuslogin
Other Setup:    Git Webhook, SonarQube Webhook, SonarQube Quality Gate, Slack configuration, Git Repository

# Slack Configuration: 
Email:      devopspractice@myyahoo.com
Workspace:  Accenture (accenture-3hn2465)
Channel:    devops_practices
Token:      sLxuMSHJ3uCrisWYGPZPFyow
