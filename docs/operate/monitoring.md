---
layout: default
title: Monitoring
parent: Operate
---

# Monitoring
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Monitoring in DevSecOps refers to the practice of continuously observing and analyzing an organization's IT systems, applications, and infrastructure to identify potential security issues, detect and respond to security incidents, and ensure compliance with security policies and regulations.

In DevSecOps, monitoring is a critical component of a comprehensive security strategy, allowing organizations to identify and respond to security threats quickly and effectively. Some of the key benefits of monitoring in DevSecOps include:

1. Early detection of security incidents: By continuously monitoring systems and applications, organizations can detect security incidents early on and take immediate action to remediate them.

2. Improved incident response: With real-time monitoring and analysis, organizations can respond to security incidents quickly and effectively, minimizing the impact of a potential breach.

3. Improved compliance: By monitoring systems and applications for compliance with security policies and regulations, organizations can ensure that they are meeting their security obligations.

4. Improved visibility: Monitoring provides organizations with greater visibility into their IT systems and applications, allowing them to identify potential security risks and take proactive steps to address them.

There are a variety of monitoring tools and technologies available that can be used in DevSecOps, including log analysis tools, network monitoring tools, and security information and event management (SIEM) solutions. These tools can be integrated with other DevSecOps practices, such as continuous integration and continuous deployment, to ensure that security is built into the application development lifecycle.




## Prometheus

Start the Prometheus server:

```
$ ./prometheus --config.file=prometheus.yml
```

Check Prometheus server status:


```
$ curl http://localhost:9090/-/healthy
```

Query data using PromQL:


```
http://localhost:9090/graph?g0.range_input=1h&g0.expr=up&g0.tab=0
```

## Grafana

Add Prometheus data source:


```
http://localhost:3000/datasources/new?gettingstarted
```


## Nagios

Configure Nagios server:


```
/etc/nagios3/conf.d/
```

Verify Nagios server configuration:


```
$ sudo /usr/sbin/nagios3 -v /etc/nagios3/nagios.cfg
```

## Zabbix

Configure Zabbix agent on the server: Edit the Zabbix agent configuration file /etc/zabbix/zabbix_agentd.conf to specify the Zabbix server IP address and hostname, and to enable monitoring of system resources such as CPU, memory, disk usage, and network interface. Example configuration:

```
Server=192.168.1.100
ServerActive=192.168.1.100
Hostname=web-server
EnableRemoteCommands=1
UnsafeUserParameters=1
# Monitor system resources
UserParameter=cpu.usage[*],/usr/bin/mpstat 1 1 | awk '/Average:/ {print 100-$NF}'
UserParameter=memory.usage,free | awk '/Mem:/ {print $3/$2 * 100.0}'
UserParameter=disk.usage[*],df -h | awk '$1 == $1 {print int($5)}'
UserParameter=network.in[*],cat /proc/net/dev | grep $1 | awk '{print $2}'
UserParameter=network.out[*],cat /proc/net/dev | grep $1 | awk '{print $10}'
```

Configure Zabbix server: Login to the Zabbix web interface and navigate to the "Configuration" tab. Create a new host with the same hostname as the server being monitored, and specify the IP address and Zabbix agent port. Add items to the host to monitor the system resources specified in the Zabbix agent configuration file. Example items:

* CPU usage: `system.cpu.util[,idle]`
* Memory usage: `vm.memory.size[available]`
* Disk usage: `vfs.fs.size[/,pfree]`
* Network inbound traffic: `net.if.in[eth0]`
* Network outbound traffic: `net.if.out[eth0]`

Configure triggers: Set up triggers to alert when any monitored item exceeds a certain threshold. For example, set a trigger on the CPU usage item to alert when the usage exceeds 80%.

Configure actions: Create actions to notify relevant stakeholders when a trigger is fired. For example, send an email to the web application team and the system administrators.


## Datadog

Edit the Datadog agent configuration file `/etc/datadog-agent/datadog.yaml` and add the following lines:

```
# Collect CPU metrics
procfs_path: /proc
cpu_acct: true

# Collect memory metrics
meminfo_path: /proc/meminfo
```

To view CPU and memory metrics, go to the Datadog Metrics Explorer and search for the metrics `system.cpu.usage` and `system.mem.used`.



Here are some sample commands you can use to collect CPU and memory metrics with Datadog:

To collect CPU metrics:


```
curl -X POST -H "Content-type: application/json" -d '{
    "series": [
        {
            "metric": "system.cpu.usage",
            "points": [
                [
                    '"$(date +%s)"',
                    "$(top -bn1 | grep '%Cpu(s)' | awk '{print $2 + $4}')"
                ]
            ],
            "host": "my-host.example.com",
            "tags": ["environment:production"]
        }
    ]
}' "https://api.datadoghq.com/api/v1/series?api_key=<YOUR_API_KEY>"
```


To collect memory metrics:


```
curl -X POST -H "Content-type: application/json" -d '{
    "series": [
        {
            "metric": "system.mem.used",
            "points": [
                [
                    '"$(date +%s)"',
                    "$(free -m | awk '/Mem:/ {print $3}')"
                ]
            ],
            "host": "my-host.example.com",
            "tags": ["environment:production"]
        }
    ]
}' "https://api.datadoghq.com/api/v1/series?api_key=<YOUR_API_KEY>"
```

Note that these commands assume that you have the necessary tools (`top`, `free`) installed on your system to collect CPU and memory metrics. You can customize the `metric`, `host`, and `tags` fields as needed to match your setup.




## New Relic

To install the New Relic Infrastructure agent on a Ubuntu server:


```
curl -Ls https://download.newrelic.com/infrastructure_agent/linux/apt | sudo bash
sudo apt-get install newrelic-infra
sudo systemctl start newrelic-infra
```

To install the New Relic Infrastructure agent on a CentOS/RHEL server:


```
curl -Ls https://download.newrelic.com/infrastructure_agent/linux/yum/el/7/x86_64/newrelic-infra.repo | sudo tee /etc/yum.repos.d/newrelic-infra.repo
sudo yum -y install newrelic-infra
sudo systemctl start newrelic-infra
```

To view CPU and memory metrics for a specific server using the New Relic API:

```
curl -X GET 'https://api.newrelic.com/v2/servers/{SERVER_ID}/metrics/data.json' \
     -H 'X-Api-Key:{API_KEY}' \
     -i \
     -d 'names[]=System/CPU/Utilization&values[]=average_percentage' \
     -d 'names[]=System/Memory/Used/Bytes&values[]=average_value' \
     -d 'from=2022-05-01T00:00:00+00:00&to=2022-05-10T00:00:00+00:00'
```




## AWS CloudWatch


1- To install the CloudWatch agent on Linux, you can use the following commands:

```
curl https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm -O
sudo rpm -i amazon-cloudwatch-agent.rpm
```

2- Configure the CloudWatch Agent to Collect Metrics


On Linux, you can create a configuration file at `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json` with the following content:


```
{
    "metrics": {
        "namespace": "CWAgent",
        "metricInterval": 60,
        "append_dimensions": {
            "InstanceId": "${aws:InstanceId}"
        },
        "metrics_collected": {
            "cpu": {
                "measurement": [
                    "cpu_usage_idle",
                    "cpu_usage_iowait",
                    "cpu_usage_user",
                    "cpu_usage_system"
                ],
                "metrics_collection_interval": 60,
                "totalcpu": false
            },
            "memory": {
                "measurement": [
                    "mem_used_percent"
                ],
                "metrics_collection_interval": 60
            }
        }
    }
}
```


On Windows, you can use the CloudWatch Agent Configuration Wizard to create a configuration file with the following settings:


```
- Choose "AWS::EC2::Instance" as the resource type to monitor
- Choose "Performance counters" as the log type
- Select the following counters to monitor:
  - Processor Information -> % Processor Time
  - Memory -> % Committed Bytes In Use
- Set the metric granularity to 1 minute
- Choose "CWAgent" as the metric namespace
- Choose "InstanceId" as the metric dimension
```

3- Start the CloudWatch Agent
Once you've configured the CloudWatch agent, you can start it on the EC2 instance using the following commands:

```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
sudo service amazon-cloudwatch-agent start
```

4- View the Metrics in CloudWatch

After a few minutes, the CloudWatch agent will start collecting CPU and memory metrics from the EC2 instance. You can view these metrics in the CloudWatch console by following these steps:

* Go to the CloudWatch console and select "Metrics" from the left-hand menu
* Under "AWS Namespaces", select "CWAgent"
* You should see a list of metrics for the EC2 instance you are monitoring, including CPU and memory usage. You can select individual metrics to view graphs and set up alarms based on these metrics.


## Azure Monitor


1- Configure the agent to collect CPU and memory metrics by adding the following settings to the agent's configuration file:


```
    {
      "metrics": {
        "performance": {
          "collectionFrequencyInSeconds": 60,
          "metrics": [
            {
              "name": "\\Processor(_Total)\\% Processor Time",
              "category": "Processor",
              "counter": "% Processor Time",
              "instance": "_Total"
            },
            {
              "name": "\\Memory\\Available Bytes",
              "category": "Memory",
              "counter": "Available Bytes",
              "instance": null
            }
          ]
        }
      }
    }
```

2- Restart the Azure Monitor agent to apply the new configuration.

3- Select the virtual machine or server that you want to view metrics for.
4- Select the CPU and memory metrics that you want to view.
5- Configure any alerts or notifications that you want to receive based on these metrics.

To collect CPU and memory metrics using Azure Monitor, you can also use the Azure Monitor REST API or the Azure CLI. Here's an example Azure CLI command to collect CPU and memory metrics:



```
az monitor metrics list --resource {resource_id} --metric-names "\Processor(_Total)\% Processor Time" "Memory\Available Bytes" --interval PT1M --start-time 2022-05-20T00:00:00Z --end-time 2022-05-21T00:00:00Z
```

This command retrieves CPU and memory metrics for a specific resource (identified by `{resource_id}`) over a one-day period (from May 20, 2022 to May 21, 2022), with a one-minute interval. You can modify the parameters to retrieve different metrics or time ranges as needed.





## Google Cloud Monitoring

1- Install the Stackdriver agent on the GCE instance. You can do this using the following command:

```
curl -sSO https://dl.google.com/cloudagents/install-monitoring-agent.sh
sudo bash install-monitoring-agent.sh
```

2- Verify that the Monitoring Agent is running by checking its service status:


```
sudo service stackdriver-agent status
```

3- In the Google Cloud Console, go to Monitoring > Metrics Explorer and select the `CPU usage` metric under the `Compute Engine VM Instance` resource type. Set the aggregation to `mean` and select the GCE instance that you created and `Click Create` chart to view the CPU usage metric for your instance.


4- To collect memory metrics, repeat step 5 but select the `Memory usage` metric instead of `CPU usage`.


## Netdata

* In the Netdata web interface, go to the "Dashboard" section and select the "system.cpu" chart to view CPU usage metrics. You can also select the "system.ram" chart to view memory usage metrics.

* To reduce failover using machine learning, you can configure Netdata's anomaly detection feature. In the Netdata web interface, go to the "Anomaly Detection" section and select "Add alarm".

* For the "Detect" field, select "cpu.system". This will detect anomalies in the system CPU usage.

* For the "Severity" field, select "Warning". This will trigger a warning when an anomaly is detected.

* For the "Action" field, select "Notify". This will send a notification when an anomaly is detected.

* You can also configure Netdata's predictive analytics feature to predict when a system will fail. In the Netdata web interface, go to the "Predict" section and select "Add algorithm".

* For the "Algorithm" field, select "Autoregression". This will use autoregression to predict system behavior.

* For the "Target" field, select "cpu.system". This will predict CPU usage.

* For the "Window" field, select "30 minutes". This will use a 30-minute window to make predictions.

* Finally, click "Create" to create the algorithm.


## Sysdig


- [ ] Capture system events and write them to a file.

```
sysdig -w <filename.scap>
```

- [ ] Customize the output format of captured events

```
sysdig -p "%evt.num %evt.type %evt.args"
```

- [ ] Filter events by process name (e.g., nginx)

```
sysdig proc.name=nginx
```

- [ ] Read events from a file and filter by process name (e.g., httpd).

```
sysdig -r <filename.scap> proc.name=httpd
```


- [ ] Display file open events

```
sysdig -c file_open
```

- [ ] Customize the output format of captured events

```
sysdig -c fdbytes_by fd.sport
```


- [ ] Monitor IP traffic in real-time.

```
sysdig -c spy_ip
```

- [ ] Show top containers by CPU usage.

```
sysdig -c topcontainers_cpu
```

- [ ] Display process execution time.

```
sysdig -c proc_exec_time
```

- [ ] Monitor system calls made by processes.

```
sysdig -c proc_calls
```

- [ ] Top Container

```
sysdig -c container_top
```

- [ ] Customize the output format of captured events

```
Show top containers by resource usage.
```

- [ ] Display Kubernetes pod information.

```
sysdig -c k8s.pods
```

- [ ] Monitor Kubernetes deployment events.

```
sysdig -c k8s.deployments
```


## Dynatrace



- [ ] Retrieve average CPU usage timeseries data for a specific time range.
Create Custom Alerting Profile:

```
timeseriesquery "metric=CPU|avg:system.cpu.usage" --start-time="2023-05-01T00:00:00Z" --end-time="2023-05-02T00:00:00Z": 
```

- [ ] Create a new alerting profile for detecting high memory usage, with a threshold of 80%.
Retrieve Deployment Events

```
create-alerting-profile --name="High Memory Usage" --metric="memory.resident" --condition="> threshold:80" --enabled=true: 
```

- [ ] Retrieve a list of deployment events that occurred within a specific time range.

```
deployment-events --start-time="2023-05-01T00:00:00Z" --end-time="2023-05-02T00:00:00Z"
```

- [ ] Create a new custom dashboard with a 2x2 layout.

```
dashboard create --name="My Custom Dashboard" --layout="2x2": 
```

- [ ] Analyze the performance and dependencies of a specific application named "My Application".

```
application analyze --name="My Application": 
```




## Alerta

### Send a new alert

Create and send a new alert to the Alerta system

```
curl -X POST -H "Content-Type: application/json" -d '{
  "resource": "webserver1",
  "event": "High CPU Usage",
  "environment": "Production",
  "severity": "major",
  "service": ["Web Servers"],
  "text": "High CPU usage detected on webserver1"
}' https://your-alerta-url/api/alert
```



### Query alerts

Retrieve alerts based on specific criteria


```
curl -X GET "https://your-alerta-url/api/alert?status=open&severity=major"
```




### Update an alert

Update the details or status of an existing alert


```
curl -X PUT -H "Content-Type: application/json" -d '{
  "status": "ack",
  "note": "Investigating the issue..."
}' https://your-alerta-url/api/alert/<alert_id>
```



### Delete an alert

Delete an existing alert from the Alerta system

```
curl -X DELETE https://your-alerta-url/api/alert/<alert_id>
```



### Get alert history

Retrieve the history of changes for a specific alert.


```
curl -X GET https://your-alerta-url/api/alert/<alert_id>/history
```



## ChatOps

### Element

#### Creating a New Matrix Account

```
# Riot
riot-web

# Element
element-web
```

#### Joining a Matrix Chat Room

```
# Riot
riot-web --url "https://matrix.org" --room "room_id"

# Element
element-web --url "https://matrix.org" --room "room_id"
```

#### Sending a Message in a Matrix Chat Room

```
# Riot
riot-web --url "https://matrix.org" --room "room_id" --message "Hello, World!"

# Element
element-web --url "https://matrix.org" --room "room_id" --message "Hello, World!"
```

#### Displaying Room Details in Matrix

```
# Riot
riot-web --url "https://matrix.org" --room "room_id" --details

# Element
element-web --url "https://matrix.org" --room "room_id" --details
```

#### Creating a New Matrix User

```
# Riot
riot-web --url "https://matrix.org" --register --username "new_user" --password "password"

# Element
element-web --url "https://matrix.org" --register --username "new_user" --password "password"
```

#### Send a deployment notification to a chat room

```
# Riot
riot-web --url "https://matrix.org" --room "room_id" --message "Deployment successful!"

# Element
element-web --url "https://matrix.org" --room "room_id" --message "Deployment successful!"
```


#### Trigger a CI/CD pipeline from a chat room

```
# Riot
riot-web --url "https://matrix.org" --room "room_id" --message "!pipeline deploy"

# Element
element-web --url "https://matrix.org" --room "room_id" --message "!pipeline deploy"
```

#### Execute a command on a remote server from a chat room

```
# Riot
riot-web --url "https://matrix.org" --room "room_id" --message "!exec ssh user@server 'ls -l'"

# Element
element-web --url "https://matrix.org" --room "room_id" --message "!exec ssh user@server 'ls -l'"
```


### Slack

#### Send a deployment notification to a Slack channel:

```
slackcli --channel "#channel_name" --message "Deployment successful!"
```

#### Trigger a CI/CD pipeline from a Slack channel:

```
slackcli --channel "#channel_name" --message "!pipeline deploy"
```

#### Execute a command on a remote server from a Slack channel:

```
slackcli --channel "#channel_name" --message "!exec ssh user@server 'ls -l'"
```

#### Request a status update from an external service in a Slack channel:

```
slackcli --channel "#channel_name" --message "!status check"
```

#### Create a new ticket in a ticketing system from a Slack channel:

```
slackcli --channel "#channel_name" --message "!ticket create 'New issue: Need assistance'"
```




## Robusta

To set custom tolerations or a nodeSelector update your generated_values.yaml file as follows:


```
global_config:
  krr_job_spec:
    tolerations:
    - key: "key1"
      operator: "Exists"
      effect: "NoSchedule"
    nodeSelector:
      nodeName: "your-selector
```



## Sensu


### Register a new check in Sensu:


```
sensuctl check create mycheck --command "check_mycheck.sh" --subscriptions linux --handlers default
```


### Register a new check in Sensu:


```
sensuctl check create mycheck --command "check_mycheck.sh" --subscriptions linux --handlers default
```


### Create a new handler in Sensu:



```
sensuctl handler create myhandler --type pipe --command "myhandler.sh"
```


### Create a new asset in Sensu:



```
sensuctl asset create myasset --url https://example.com/myasset.tar.gz --sha512sum abcdef1234567890
```


### Create a new namespace in Sensu:



```
sensuctl namespace create mynamespace
```




### Create a new filter in Sensu:



```
sensuctl filter create myfilter --action allow --expressions "event.Entity.Environment == 'production'"
```




## Steampipe


### Check for open security groups in AWS

```
select
    aws_vpc.vpc_id,
    aws_security_group.group_id,
    aws_security_group.group_name,
    aws_security_group.description
from
    aws_security_group
    inner join aws_vpc on aws_security_group.vpc_id = aws_vpc.vpc_id
where
    aws_security_group.security_group_status = 'active'
    and aws_security_group.group_name != 'default'
    and aws_security_group.ip_permissions_egress = '0.0.0.0/0'
```


### Check for public S3 buckets in AWS

```
select
    aws_s3_bucket.bucket_name,
    aws_s3_bucket.creation_date,
    aws_s3_bucket.owner_id,
    aws_s3_bucket.owner_display_name
from
    aws_s3_bucket
where
    aws_s3_bucket.acl = 'public-read' or aws_s3_bucket.acl = 'public-read-write'
```


### Check for unencrypted RDS instances in AWS

```
select
    aws_rds_db_instance.db_instance_identifier,
    aws_rds_db_instance.encrypted,
    aws_rds_db_instance.engine,
    aws_rds_db_instance.engine_version
from
    aws_rds_db_instance
where
    aws_rds_db_instance.encrypted = false
```


### Check for outdated Docker images in Docker Hub

```
select
    docker_hub_image.namespace,
    docker_hub_image.name,
    docker_hub_image.tag,
    docker_hub_image.image_created
from
    docker_hub_image
where
    docker_hub_image.image_created < date_sub(current_date, interval 30 day)
```




### Check for unused IAM access keys in AWS

```
select
    aws_iam_access_key.access_key_id,
    aws_iam_access_key.user_name,
    aws_iam_access_key.create_date,
    aws_iam_access_key.status
from
    aws_iam_access_key
where
    aws_iam_access_key.status = 'Active'
    and not exists (
        select
            1
        from
            aws_iam_user
        where
            aws_iam_user.user_name = aws_iam_access_key.user_name
            and aws_iam_user.user_enabled = true
    )
```




## Sysdig



### Capture system activity and save it to a file for later analysis  

```
sysdig -w <output_file>
```


### Display a live system activity summary with top processes 

```
sysdig -c top
```

### Monitor network activity with a summary of network connections  

```
sysdig -c netstat
```

### Filter system activity based on process name  

```
sysdig proc.name=<process_name>
```

### Filter system activity based on process ID (PID)  

```
sysdig proc.pid=<process_id>
```

### Monitor disk I/O activity for a specific process  

```
sysdig -p"%proc.name %evt.type" fd.type=char fd.name=/dev/sdX
```

### Trace system calls made by a specific process

```
sysdig -p"%proc.name %evt.type %evt.args" proc.name=<process_name>
```

### Monitor file system activity within a directory

```
sysdig -p"%evt.type %evt.args" evt.dir=<directory_path>
```

### Monitor system calls related to process creation

```
sysdig -p"%proc.name %evt.type" evt.type=clone or evt.type=fork
```


## Sysdig Inspect


### Launch Sysdig Inspect on a live running container 

```
sysdig -p"%proc.name %evt.type" evt.type=clone or evt.type=fork
```

### Launch Sysdig Inspect on a specific trace file for offline analysis 

```
sysdig-inspect trace <trace_file>
```

### Filter the displayed events based on a specific process 

```
filter proc.name=<process_name>
```

### Filter the displayed events based on a specific system call 

```
filter evt.type=<system_call>
```

### Inspect the file system events within a specific directory  

```
fs.directory=<directory_path>
```

### Inspect the system calls made by a process  

```
syscall <process_name>
```

### Inspect the network connections of a process  

```
netconn <process_name>
```

### Inspect the open file descriptors of a process  

```
openfiles <process_name>
```

### Display a summary of captured system calls and events 

```
events
```


## Monitoring cron files  


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_cron.py



## Monitoring /etc/hosts file 


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_hosts_file.py


## Monitoring /etc/ld.so.preload file 


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_ld_preload.py


## Monitoring /etc/passwd file  


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_passwd.py


## Monitoring modules 


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_modules.py


## Monitoring SSH authorized_keys files 


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_ssh_authorized_keys.py


## Monitoring systemd unit files  


https://github.com/sqall01/LSMS/blob/main/scripts/monitor_systemd_units.py


## Search executables in /dev/shm 


https://github.com/sqall01/LSMS/blob/main/scripts/search_dev_shm.py


## Search fileless programs (memfd_create)    


https://github.com/sqall01/LSMS/blob/main/scripts/search_memfd_create.py


## Search hidden ELF files  


https://github.com/sqall01/LSMS/blob/main/scripts/search_hidden_exe.py



## Search immutable files  


https://github.com/sqall01/LSMS/blob/main/scripts/search_immutable_files.py




## Search kernel thread impersonations  


https://github.com/sqall01/LSMS/blob/main/scripts/search_non_kthreads.py



## Search processes that were started by a now disconnected SSH session  


https://github.com/sqall01/LSMS/blob/main/scripts/search_ssh_leftover_processes.py




## Search running deleted programs   


https://github.com/sqall01/LSMS/blob/main/scripts/search_deleted_exe.py



## Test script to check if alerting works   


https://github.com/sqall01/LSMS/blob/main/scripts/test_alert.py



## Verify integrity of installed .deb packages   


https://github.com/sqall01/LSMS/blob/main/scripts/verify_deb_packages.py

