# Case Study 3 for DS553 - Group 16 - Documentation

## Overview: Container Names & Ports
### Container Domain Names: all on Docker network 'monitoring'
Product Container Domain Name: `group16-recipebot`  
Prometheus Container Domain Name: `group16-prometheus`  
Grafana Container Domain Name: `group16-grafana`  
### External Ports
`7000`: Chatbot (Internal: `7860`)  
`7001`: Prometheus metrics (Internal: `8000`)  
`7002`: Node Exporter metrics (Internal: `9100`)  
`7003`: Grafana   

## Part 1 - Deployment of Products on Docker Containers on the Shared VM - `app.py`, Dockerfile
### Local & API Based Product
#### Modifications made to `app.py`:
1. We added Prometheus metrics. These were similar to what was implemented in the Professor's example case study 3. However, we made changes so that we can later seperate out the metrics by our local model vs our API based model on Grafana. We will also still be able to sum these across both for overall metrics of the entire app, regardless of model the user selected. These metrics were implemented in code at the top of the `app.py` file as well as counters added in their respective sections that made sense for the specific metric.
2. We changed the the last `if__name__ == "__main__"` function to specify prometheus metrics to start on port `8000`.
#### Modificiations made to `requirements.txt`:
We updated our requirements to get rid of packages we are no longer using such as `nvidia-ml-py` and `psutil`. We also added prometheus-client to be able to expose our product's metrics to Prometheus.
#### Dockerfile specifications:
We followed the Professor's Dockerfile example and similarly exposed `7860`, `8000`, and `9100` for our internal ports. We also kept the command `RUN pip install --no-cache-dir -r /opt/app/requirements.txt` to install our requirements for the app to run.
#### Steps to create Docker container:
After creating our Github repo with the initial versions of `app.py`, `requirements.txt`, and Dockerfile, we cloned this onto the group16 account on the shared VM into the folder `case_study_3`. Then, we first built our Docker image with the docker build command, then we created and started our product container with the Docker run command. We created our container on the monitoring network so it would be easy to connect it to our other Prometheus and Grafana containers. We also mapped our internal ports `7860`, `8000`, and `9100` to allow for access from outside the monitoring network, with `-p 7000:7860 -p 7001:8000 and -p 7002:9100`. We used the ssh tunnel command from class to test our models before building an ngrok link later. They both worked and our Gradio app was accessible from a local http link. When I updated the Prometheus query metrics, I created a new Docker image with the same commands called `group16-recipebot:v2`, so this is the most up to date image for the product. 
##### Selected port range in Google spreadsheet: `7000-7100`

## Part 2 - Monitoring Our Products - Prometheus Folder -> `prometheus.yml`, Dockerfile
### Prometheus Container Implementation
Following the Professor's Prometheus implementation examples, we created a Prometheus folder. This contains `prometheus.yml` and Dockerfile. Our `prometheus.yml` file tells Prometheus that our app is accessible through the product container's internal port `8000` for query level metrics, and that for Node Exporter system level metrics, it can get these from internal port `9100`. In our Dockerfile in our Prometheus folder, we copied over the official Prometheus Docker image from the Docker Hub. We also exposed internal port `9090` so that later Grafana can query this port for our app and system level metrics. We pulled our github repo again onto the group16 account on the VM to now include our Prometheus folder. From there, we ran the docker build command to create our Prometheus Docker image. We then ran the docker run command, specifying for this container to also be on the monitoring network. Because our Prometheus container is on the same monitoring network as the app container and the Grafana container, we did not need to use the `-p` Docker argument. Instead, Grafana will be able to query Prometheus through its internal port `9090` as they are on the same monitoring network.
### Testing Prometheus Works
We tested Prometheus was successfully collecting our metrics by running a few chatbot recipe requests on our open app (through an open ssh tunnel). We then made an http request through a `docker exec group16-prometheus wget` command to verify that Prometheus was successfully scraping our app container. We also made two other similar requests to verify that we could seperate local from remote model requests and that Node Exporter was able to retrieve the system metrics successfully.

## Part 3 - Exposing Our Products Globally - ngrok Folder -> `ngrok.yml`, `ngrok-grafana.yml`
### ngrok Container Implementation
For our receipebot to be reachable outside of the VM, we ran ngrok as a Docker container on the same monitoring network as our other containers. Since ngrok is on the monitoring network, it reaches the product through the container name and the internal port (`group16-recipebot:7860`) instead of the externally mapped port 7000. We created a ngrok folder that contains the configuration file `ngrok.yml` that defines a single endpoint pointing to the local http link of the recipebot and uses the free static domain from one of our ngrok accounts. We mounted this file into the container with the flag `-v` and modifier `:ro` at the end of the file path instead of copying it into the image and making the file read-only, so our authtoken is never in the image itself and the file can't be changed or deletd by the container. We also added the ngrok config files to `.gitignore` so the token is not pushed to GitHub. We ran the container with `--restart unless-stopped` so that the tunnel comes back if the container or VM restarts.
#### Steps to create the ngrok container:
We tagged the official image as `group16-ngrok:v1`, then created the container with `docker run -d --name group16-ngrok --network monitoring --restart unless-stopped -v <path to ngrok.yml>:/etc/ngrok.yml:ro group16-ngrok:v1 start --all --config /etc/ngrok.yml --log stdout`. 
### Second ngrok Container Implementation
Since the free ngrok tier only gives each account one domain, two endpoints in the same configuration file ends up on the same URL. So, to expose both the recipebot and Grafana at the same time, we used a second ngrok account. It runs in a separate container, `group16-ngrok-grafana`, with its own configuration file  `ngrok-grafana.yml` containing one endpoint to the local http link of Grafana.
#### Steps to create the ngrok-grafana container:
We tagged the official image as `group16-ngrok-grafana:v1`, then created the container with `docker run -d   --name group16-ngrok-grafana   --network monitoring   --restart unless-stopped   -v <path to ngrok-grafana.yml>:/etc/ngrok.yml:ro   ngrok/ngrok:latest start --all --config /etc/ngrok.yml --log stdout`. 
### Public URLs
What's For Dinner?: https://untracked-unwomanly-untrained.ngrok-free.dev

Grafana: https://crewless-basics-idealness.ngrok-free.dev

Login for Grafana -

username: public

password: 1324589

## Part 4 - Visualizing Metrics with Grafana - Grafana Folder -> `grafana.yml`, Dockerfile
### Grafana Container Implementation
We created a Grafana folder that contains the provisioning file `grafana.yml` and Dockerfile. The Dockerfile starts from the official Grafana image and copies `grafana.yml` into Grafana's data source provisioning directory, so the Prometheus data source is created automatically on startup instead of by hand in the UI. The data source points to local http link of Prometheus with proxy ("Server") access, which works because Grafana and Prometheus are both on the monitoring network.
#### Steps to create the Grafana container:
We built the image with `docker build -t group16-grafana:v1 ./grafana`, then ran it on the monitoring network with `-p 7003:3000`. We mounted the `group16-grafana-data` volume at `/var/lib/grafana` so dashboards survive container re-creation, and we set an admin password because the dashboard is public.
### Dashboard
Our dashboard `🍽️ What's For Dinner? Metrics` includes: `Overview Row (Total Requests Made, Total Failure Requests, Success Rate), Local vs API Comparison Row (Request Count by Model, Average Latency by Model, Failure Requests by Model) and System (CPU %, RAM %)`. We created a user that can be used by anyone to view the dashboard. The login details are under the Grafana public URL.



