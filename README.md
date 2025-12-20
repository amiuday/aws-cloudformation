User-data script

#!/bin/bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.329.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.329.0/actions-runner-linux-x64-2.329.0.tar.gz

echo "194f1e1e4bd02f80b7e9633fc546084d8d4e19f3928a324d512ea53430102e1d  actions-runner-linux-x64-2.329.0.tar.gz" | shasum -a 256 -c

tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz

./config.sh --url https://github.com/amiuday/aws-cloudformation --token AQATMFTFN3MPYJZSDHJPJRDJIOYFU

 nohup ./run.sh > runner.log 2>&1 &

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install


# Docker Installation 
For Amazon Linux 2
sudo yum install docker -y

(If Amazon Linux 2023)
sudo dnf install docker -y

🔹 Step 4: Start Docker service
sudo systemctl start docker
sudo systemctl enable docker


Check status:

sudo systemctl status docker


You should see active (running) ✅
🔹 Step 5: Allow ec2-user to run Docker (IMPORTANT)

Without this, you’ll need sudo every time.

sudo usermod -aG docker ec2-user


⚠️ Logout and login again

exit


Reconnect via SSH.
🔹 Step 6: Verify Docker installation
docker --version


Then run:

docker run hello-world

Expected output:

Hello from Docker!
This message shows that your installation appears to be working correctly.

🎉 If you see this → Docker is READY.








===========================================================================
#  Node_exporter Installation

sudo yum update -y
sudo useradd --no-create-home node_exporter
cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
sudo tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo chown -R node_exporter:node_exporter node_exporter-1.8.2.linux-amd64

sudo vi /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/opt/node_exporter-1.8.2.linux-amd64/node_exporter

[Install]
WantedBy=default.target

sudo systemctl daemon-reexec
sudo systemctl enable node_exporter
sudo systemctl start node_exporter


# Install Prometheus
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
tar xvf prometheus-2.53.0.linux-amd64.tar.gz

vi /opt/prometheus-2.53.0.linux-amd64/prometheus.yml
global:
 scrape_interval: 15s

scrape_configs:
 - job_name: "node"
   static_configs:
     - targets: ["localhost:9100"]

nohup /opt/prometheus-2.53.0.linux-amd64/prometheus \
            --config.file=/opt/prometheus-2.53.0.linux-amd64/prometheus.yml &

# Install Grafana
yum install -y grafana
No match for argument: grafana Error: Unable to find a match: grafana

sudo tee /etc/yum.repos.d/grafana.repo <<'EOF'
[grafana]
name=Grafana OSS
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF

sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server

Expected:
Active: active (running)


=============================================================================
To monitor another EC2, you need:
1️⃣ Node Exporter running on that EC2 - same as above
2️⃣ Prometheus able to reach port 9100 (App EC2 SG → inbound 9100 only from Monitoring SG)
3️⃣ Prometheus config updated with that target
sudo vi /opt/prometheus-2.53.0.linux-amd64/prometheus.yml

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "ec2-nodes"
    static_configs:
      - targets:
          - "localhost:9100"
          - "PRIVATE-IP-OF-OTHER-EC2:9100"

⚠️ Use private IP (best inside VPC).

4️⃣ Prometheus reloaded
sudo systemctl reload prometheus

If reload is not configured:
sudo systemctl restart prometheus

Go to:
Status → Targets

==========================================================================================
Where you run queries
Open Prometheus UI:

http://<PROMETHEUS-IP>:9090


Go to the Graph tab.
This is where all PromQL starts.

0️⃣ First sanity query (ALWAYS do this)
up

What it shows

1 → target is UP

0 → target is DOWN

This is the health check query.

1️⃣ CPU usage queries
Total CPU usage (%)
100 - avg by(instance)(
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100


📌 Real meaning:

“How busy is each EC2 on average over last 5 minutes?”


CPU usage per core
100 - avg by(instance, cpu)(
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100


Use this when:

One core is overloaded

Others are idle
