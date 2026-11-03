# Case Study 3 for DS553 - Group 16 - Documentation

## Overview: Container Names & Ports
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
After creating our Github repo with the initial versions of app.py, requirements.txt, and Dockerfile, we cloned this onto the group16 account on the shared VM into the folder case_study_3. Then, we first built our Docker image with the docker build command, then we created and started our product container with the Docker run command. We created our container on the monitoring network so it would be easy to connect it to our other Prometheus and Grafana containers. We also mapped our internal ports 7860, 8000, and 9100 to allow for access from outside the monitoring network, with -p 7000:7860 -p 7001:8000 and -p 7002:9100. We used the ssh tunnel command from class to test our models before building an ngrok link later. They both worked and our Gradio app was accessible from a local http link. When I updated the Prometheus query metrics, I created a new Docker image with the same commands called group16-recipebot:v2, so this is the most up to date image for the product. 
##### Selected port range in Google spreadsheet: 7000-7100

## Part 2 - Monitoring Our Products - Prometheus Folder -> prometheus.yml, Dockerfile
### Prometheus Container Implementation
Following the Professor's Prometheus implementation examples, we created a Prometheus folder. This contains prometheus.yml and Dockerfile. Our prometheus.yml file tells Prometheus that our app is accessible through the product container's internal port 8000 for query level metrics, and that for Node Exporter system level metrics, it can get these from internal port 9100. In our Dockerfile in our Prometheus folder, we copied over the official Prometheus Docker image from the Docker Hub. We also exposed internal port 9090 so that later Grafana can query this port for our app and system level metrics. We pulled our github repo again onto the group16 account on the VM to now include our Prometheus folder. From there, we ran the docker build command to create our Prometheus Docker image. We then ran the docker run command, specifying for this container to also be on the monitoring network. Because our Prometheus container is on the same monitoring network as the app container and the Grafana container, we did not need to use the -p Docker argument. Instead, Grafana will be able to query Prometheus through its internal port 9090 as they are on the same monitoring network.
### Testing Prometheus Works
We tested Prometheus was successfully collecting our metrics by running a few chatbot recipe requests on our open app (through an open ssh tunnel). We then made an http request through a docker exec group16-prometheus wget command to verify that Prometheus was successfully scraping our app container. We also made two other similar requests to verify that we could seperate local from remote model requests and that Node Exporter was able to retrieve the system metrics successfully.
