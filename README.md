# Assignement3
Create a local VM and implement a mechanism to monitor resource usage. Configure it to auto-scale to a public cloud (e.g., GCP, AWS, or Azure) when resource usage exceeds 75%.

SCRIPT:
# Script to automate the creation of a local VM, resource monitoring setup, and auto-scaling to GCP when resource usage exceeds 75%

# Step 1: Install and configure Prometheus and Grafana for resource monitoring on local VM

# Update system and install dependencies
echo "Updating system and installing dependencies..."
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y wget curl software-properties-common apt-transport-https

# Install Prometheus
echo "Installing Prometheus..."
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

# Download and install Prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.30.3/prometheus-2.30.3.linux-amd64.tar.gz
tar xvf prometheus-2.30.3.linux-amd64.tar.gz
sudo mv prometheus-2.30.3.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.30.3.linux-amd64/promtool /usr/local/bin/

# Configure Prometheus
sudo cp prometheus-2.30.3.linux-amd64/prometheus.yml /etc/prometheus/

# Set permissions
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chmod -R 755 /etc/prometheus /var/lib/prometheus

# Start Prometheus
echo "Starting Prometheus..."
sudo nohup prometheus --config.file=/etc/prometheus/prometheus.yml &

# Install Grafana for visualizing metrics
echo "Installing Grafana..."
sudo apt-get install -y grafana

# Start Grafana
echo "Starting Grafana..."
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

# Step 2: Set up Prometheus to monitor local resources using node_exporter
echo "Setting up Prometheus Node Exporter for local system metrics..."
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvf node_exporter-1.3.1.linux-amd64.tar.gz
sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/

# Start Node Exporter
echo "Starting Node Exporter..."
nohup /usr/local/bin/node_exporter &

# Step 3: Configure Google Cloud VM (GCP) for auto-scaling
echo "Creating Google Cloud VM instance..."

# Authenticate with GCP
gcloud auth login
gcloud config set project qwiklabs-gcp-03-616c40de2296

# Create a new Google Cloud instance (use your VM configuration)
gcloud compute instances create "m23aid009" \
  --zone="us-west1-a" \
  --image-family="debian-11" \
  --image-project="debian-cloud" \
  --machine-type="n1-standard-1" \
  --tags=http-server,https-server

# Install and configure Google Cloud Monitoring Agent for auto-scaling
echo "Installing Google Cloud Monitoring Agent on GCP instance..."
gcloud compute ssh "m23aid009" --zone="us-west1-a" --command="sudo apt-get install -y google-cloud-monitoring-agent"

# Step 4: Set up Auto-Scaling on GCP when resource usage exceeds 75%
echo "Setting up Auto-scaling policy in GCP..."

# Enable auto-scaling for the instance group
gcloud compute instance-groups managed set-autoscaling "m23aid009-group" \
  --zone="us-west1-a" \
  --min-num-replicas=1 \
  --max-num-replicas=5 \
  --target-cpu-utilization=0.75

# Step 5: Deploy a simple application to test auto-scaling
echo "Deploying sample application..."

# Create a simple Python Flask application (example)
echo "from flask import Flask" > app.py
echo "app = Flask(__name__)" >> app.py
echo "@app.route('/')" >> app.py
echo "def hello_world():" >> app.py
echo "    return 'Hello, world!'" >> app.py
echo "if __name__ == '__main__':" >> app.py
echo "    app.run(host='0.0.0.0', port=5000)" >> app.py

# Install required Python packages
sudo apt-get install -y python3-pip
pip3 install Flask

# Start the application
echo "Starting sample Flask application..."
nohup python3 app.py &

# Step 6: Set up monitoring and scaling to auto-scale the resources to GCP
echo "Monitoring and scaling setup is complete."

# Output completion message
echo "Local VM monitoring is set up and auto-scaling to GCP is configured successfully."

ARCHITECTURE DIAGRAM:
![output (2)](https://github.com/user-attachments/assets/f51c056d-ed3d-4939-9f74-2f61b302851c)





