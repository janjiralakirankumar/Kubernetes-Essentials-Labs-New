```
#!/bin/bash

echo "Let's get started with Kubernetes cluster creation using KOPS!"
echo "Enter your name:"
read username
lower_username=$(echo -e "$username" | sed 's/ //g' | tr '[:upper:]' '[:lower:]')
date_now=$(date "+%F-%H-%m")
clname="${lower_username}-${date_now}.k8s.local"
echo "Your Kubernetes cluster name will be $clname"

TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") 

az=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone)
region=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/region)

sudo sed -i "/\$nrconf{restart}/d" /etc/needrestart/needrestart.conf
echo "\$nrconf{restart} = 'a';" | sudo tee -a /etc/needrestart/needrestart.conf
export DEBIAN_FRONTEND=noninteractive
export NEEDRESTART_MODE=a

sudo apt update -y
sudo apt install nano curl python3-pip -y
sudo snap install aws-cli --classic

curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.25.2/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

curl -LO https://github.com/kubernetes/kops/releases/download/v1.25.2/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

ssh-keygen -t rsa -N "" -f "$HOME/.ssh/id_rsa" -q <<<y >/dev/null 2>&1

aws s3 mb "s3://$clname" --region "$region"

export KOPS_STATE_STORE="s3://$clname"

kops create cluster --node-count=2 --master-size="t2.medium" --node-size="t2.medium" --master-volume-size=20 --node-volume-size=20 --zones "$az" --name "$clname" --ssh-public-key ~/.ssh/id_rsa.pub --yes

kops update cluster "$clname" --yes
echo "export KOPS_STATE_STORE=s3://$clname" >> /home/ubuntu/.bashrc
source /home/ubuntu/.bashrc

kops export kubecfg --admin

for ((x = 0; x < 30; x++)); do
  echo "Validating Cluster"
  kops validate cluster >status.txt 2>/dev/null
  if grep -q "is ready" status.txt; then
    echo "Your Cluster is now ready!"
    break
  else
    sleep 20
    echo "x: $x"
  fi
done

kops delete cluster --name "$clname" >delete.txt
sg1=$(grep "sg-" delete.txt | awk '{print $3}' | sed -n '1p')
sg2=$(grep "sg-" delete.txt | awk '{print $3}' | sed -n '2p')
sg3=$(grep "sg-" delete.txt | awk '{print $3}' | sed -n '3p')
aws ec2 authorize-security-group-ingress --group-id "$sg1" --protocol tcp --port 30000-32767 --cidr 0.0.0.0/0 --region "$region"
aws ec2 authorize-security-group-ingress --group-id "$sg2" --protocol tcp --port 30000-32767 --cidr 0.0.0.0/0 --region "$region"
aws ec2 authorize-security-group-ingress --group-id "$sg3" --protocol tcp --port 30000-32767 --cidr 0.0.0.0/0 --region "$region"

echo "export KOPS_STATE_STORE=s3://$clname" >delete-kops.sh
echo "kops delete cluster --name $clname --yes" >>delete-kops.sh
chmod +x delete-kops.sh
chown ubuntu:ubuntu delete-kops.sh

echo "Creating Kubernetes Dashboard"

cat >kubernetes-dashboard.yaml <<EOF
<Your Kubernetes Dashboard YAML content here>
EOF

kubectl apply -f kubernetes-dashboard.yaml

echo "Retrieving URL of SVC"
url1="https://$(kubectl get nodes -o wide -n kubernetes-dashboard | awk '{print \$7}' | sed -n '2p'):32000"
url2="https://$(kubectl get nodes -o wide -n kubernetes-dashboard | awk '{print \$7}' | sed -n '3p'):32000"
url3="https://$(kubectl get nodes -o wide -n kubernetes-dashboard | awk '{print \$7}' | sed -n '4p'):32000"

{
  echo "********        HERE ARE THE DETAILS REQUIRED        ********"
  echo "******** You can use any one of the below given URLs ********"
  echo "First URL is: $url1"
  echo "Second URL is: $url2"
  echo "Third URL is: $url3"
} >token.txt

echo "Creating Token"
kubectl -n kube-system describe secret "$(kubectl -n kube-system get secret | grep kops-admin | awk '{print \$1}')" | grep token: | awk '{print \$2}' >>token.txt
echo "******************          END          ******************" >>token.txt

cat token.txt
```
