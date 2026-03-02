install Jenkins in EC2 instance

install docker https://docs.docker.com/engine/install/ubuntu/
install docker compose : sudo apt install docker-compose

provide the permission to jenkins user 

sudo usermod -aG docker jenkins
sudo systemctl restart jenkins      # or reboot the VM

After Jenkins and created deployment and service ymal

🚀 Step 1 — Login to ECR (From Your Local Machine)

Run:

aws ecr get-login-password --region us-west-1 | \
docker login --username AWS \
--password-stdin 975050024946.dkr.ecr.us-west-1.amazonaws.com

Make sure login succeeds.