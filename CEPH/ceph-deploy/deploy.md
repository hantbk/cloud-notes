# Triển khai cụm Ceph với cephadm

Node vm1: 192.168.103.148 chạy với quyền root user

## Bước 1: Cài đặt cephadm

```bash
apt install -y cephadm
```

## Bước 2: Bootstrap cephadm

```bash
cephadm bootstrap --mon-ip 192.168.103.150
```

```bash
Ceph Dashboard is now available at:

     URL: https://vm1:8443/
    User: admin
Password: uj2bt96evc

Enabling client.admin keyring and conf on hosts with "admin" label
```

Command trên sẽ :

- Tạo ra 1 Monitor và 1 Manager trên node hiện tại (node vm1)
- Tạo ra 1 SSH key pair trên ceph cluster và lưu nó vào root user của node hiện tại `/root/.ssh/authoized_keys`
- Tạo 1 bản copy của public key vào `/etc/ceph/ceph.pub`
- Tạo 1 file cấu hình đơn giản `/etc/ceph/ceph.conf`. File này cần thiết để giao tiếp với các daemon khác
- Tạo 1 bản copy của `client.admin` secret key vào `/etc/ceph/ceph.client.admin.keyring`.
- Thêm label `_admin` vào bootstrap host. Mặc định, bất kỳ host nào với label này sẽ nhận được 1 bản copy của `/etc/ceph/ceph.conf` và `/etc/ceph/ceph.client.admin.keyring` từ bootstrap host.



