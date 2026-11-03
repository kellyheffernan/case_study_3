# Case Study 3 for DS553 - Group 16 - Full Documentation

## Overview: Container Names, Ports, etc.
### Container Domain Names: all on Docker network 'monitoring'
Product Container Domain Name: group16-recipebot
Prometheus Container Domain Name: group16-prometheus
Grafana Container Domain Name: TBD

### External Ports
7000: Chatbot (Internal: 7860)
7001: Prometheus metrics (Internal: 8000)
7002: Node Exporter metrics (Internal: 9100)
7003?: TBD, placeholder for Grafana 

## Part 1 - Deployment of Products on Docker Containers on the Shared VM - app.py, Dockerfile
### Local & API Based Product
#### Modifications made to app.py:
1. We added Prometheus metrics. These were similar to what was implemented in the Professor's example case study 3. However, we made changes so that we can later seperate out the metrics by our local model vs our API based model on Grafana. We will also still be able to sum these across both for overall metrics of the entire app, regardless of model the user selected. These metrics were implemented in code at the top of the app.py file as well as counters added in their respective sections that made sense for the specific metric.
2. We changed the the last if__name__ == "__main__" function to specify prometheus metrics to start on port 8000.
#### Modificiations made to requirements.txt:
We updated our requirements to get rid of packages we are no longer using such as nvidia-ml-py and psutil. We also added prometheus-client to be able to expose our product's metrics to Prometheus.
   
#### Dockerfile specifications:
We followed the Professor's Dockerfile example and similarly exposed 7860, 8000, and 9100 for our internal ports. We also kept the command RUN pip install --no-cache-dir -r /opt/app/requirements.txt to install our requirements for the app to run.
#### Steps to create Docker container:
We first built our Docker image with the docker build command, then we created and started our product container with the Docker run command. We built this on the monitoring network so it would be easy to share our product for grading and also for connecting it to our other Prometheus and Grafana containers. We used the ssh tunnel command from class to test our models before building an ngrok link later.
Selected port range: 7000-7100

## Part 2 - Monitoring Our Products - Prometheus Folder -> prometheus.yml, Dockerfile
### 
