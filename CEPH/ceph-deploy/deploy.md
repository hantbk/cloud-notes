# Triển khai cụm Ceph với cephadm

Node vm1: 192.168.103.148

## Bước 1: Cài đặt docker, openssh, ceph trên từng cụm máy 

### Docker

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce

sudo systemctl start docker
sudo systemctl enable docker
```

### Openssh

```bash
sudo apt install openssh-server
sudo systemclt enable ssh
```

### Cephadm

```bash
apt update -y
apt install -y cephadm ceph-common
```
