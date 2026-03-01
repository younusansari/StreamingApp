install Jenkins in EC2 instance

install docker https://docs.docker.com/engine/install/ubuntu/
install docker compose : sudo apt install docker-compose

provide the permission to jenkins user 

sudo usermod -aG docker jenkins
sudo systemctl restart jenkins      # or reboot the VM