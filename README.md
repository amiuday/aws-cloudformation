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
