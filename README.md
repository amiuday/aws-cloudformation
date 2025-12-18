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
