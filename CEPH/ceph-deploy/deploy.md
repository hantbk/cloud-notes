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

```bash
nano /etc/ssh/sshd_config
PermitRootLogin yes
```

## Bước 2: Khởi tạo Cluster

Deploy ceph trên node vm1 : 192.168.103.149

```bash
cephadm bootstrap --mon-ip 192.168.103.149 --ssh-user root --cluster-network 192.168.103.0/24
```

Ta có thông tin về cluster và dashboard

Ceph Dashboard is now available at:

	     URL: https://vm2:8443/
	    User: admin
	Password: anm8m5sal4

Trước đó, cần xét iptables mở các cổng cần thiết cho các node monitor, osd, mgr,...
Ceph monitor thường mặc định lấy các gói tin trên cổng 3300 và 6789
Ceph OSD daemons thường sẽ kết nối bắt đầu từ cổng 6800, và kết nối đến giải cổng cuối là 7568. Mỗi lần kết nối trên 1 cổng không thành công, ceph OSD daemons sẽ tự động thử kết nối lên trên cổng tiếp theo.
Thay đổi luật trong iptables của các node bằng lệnh sau:

```bash
# Đảm bảo các truy cập đầu vào của Ceph ports trên subnet
iptables -A INPUT -s 192.168.1.0/24 -p tcp -m state --state NEW -m multiport --dports 3300,6789,6800:7300,9283,8888,8889,18080,9100,9222 -j ACCEPT

# Đảm bảo các truy cập đầu ra của Ceph ports trên subnet
iptables -A OUTPUT -d 192.168.1.0/24 -p tcp -m state --state NEW -m multiport --dports 3300,6789,6800:7300,9283,8888,8889,18080,9100,9222 -j ACCEPT

# Cấp quyền kết nối SSH trên subnets
iptables -A INPUT -s 192.168.1.0/24 -p tcp -m state --state NEW --dport 22 -j ACCEPT
iptables -A OUTPUT -d 192.168.1.0/24 -p tcp -m state --state NEW --dport 22 -j ACCEPT
```

Chạy tự động bằng file osd_spec.yaml

```bash
service_type: osd
service_id: osd_spec_hdd
placement:
  hosts:
    - vm2
spec:
  osds_per_device: 3
  data_devices: #Ổ chứa data
    rotational: 1 #Do là HDD nên có xoay
    size: '20G:' #Lấy tất cả các ổ kích thước lớn hơn 20
```

Chạy dry run để kiểm tra kiểm thử trước:

```bash
ceph orch apply -i osd_spec.yaml --dry-run
```


