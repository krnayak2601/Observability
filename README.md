# Observability
Uploading all the documents from Udemy, YouTube and self learning
Attended handson session from Udemy : https://amdocsglobal.udemy.com/course/prometheus-course(Learn A to Z of Prometheus & Grafana from Basic to ADVANCE level; Complete Guide to Master DevOps Monitoring & Alerting)
This course gives a clear picture for beginners on Promethues

I created an EC2 instance from AWS for my handson session
Additonal configurations to-do if you like to have handson session on an EC2
1. Create inbound rules in security group of EC2 to access all the UI - Promethues, PythonApp for Gauge,Counter etc metrictypes, Node Exporter


Folder : 
1. Node Exporter contains exporter for linux machine
2. prom_python_app contains a simple python application and sample to instrument it to get each metric









Summary Metric:
1. In this example(summary.py) the metric calculates response time i.e. difference between end and start time
2. To calculate avg latency using this divide rate of latency sum by rate of latency by count
