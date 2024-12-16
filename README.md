# workflows
Reusable workflows and actions


## How to create a self-hosted runner on OrbStack

```
orb create -a arm64 ubuntu:noble self-hosted-runner
orb shell -m self-hosted-runner
```

TODO https://github.com/actions/partner-runner-images/blob/main/images/arm-ubuntu-24-image.md

```
sudo apt-get install -y git curl wget jq

sudo groupadd runner
sudo useradd -m -g runner -s /bin/bash runner-categolj

curl -sSL https://get.docker.com/ | sudo sh
sudo usermod -aG docker runner-categolj
sudo usermod -aG docker $(id -un)
```

```
sudo su - runner-categolj

mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-arm64-2.321.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-arm64-2.321.0.tar.gz
tar xzf ./actions-runner-linux-arm64-2.321.0.tar.gz

./config.sh --url https://github.com/categolj --name orbstack --labels orbstack --runnergroup Default --work _work --token AAA********
exit


sudo su -

cd /home/runner-categolj/actions-runner 
./svc.sh install runner-categolj
./svc.sh start
./svc.sh status
```