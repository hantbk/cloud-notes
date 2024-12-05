# Triển khai cụm Ceph với cephadm

## Requirements

- Python3
- Systemd
- Podman hoặc Docker
- Time synchronization (Chrony hoặc `ntpd`)
- LVM2

Node vm1: 192.168.103.150 chạy với quyền root user

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

## Further information about ceph bootstrap

Bootstrap mặc định sẽ hoạt động cho phần lớn người dùng:

- Mặc định, Ceph daemons gửi log tới stdout/stderr, được container runtime (docker hoặc podman) và gửi đến journald. Nếu muốn Ceph ghi log truyền thống đến `/var/log/ceph/$fsid`, cần thêm `--log-to-file` vào câu lệnh bootstrap.
- Những cụm Ceph hoạt động tốt nhất khi public network và cluster network là 2 mạng riêng biệt. Internal Cluster chịu trách nhiệm replication, recovery và heartbeats giữa các OSD daemons. Có thể định nghĩa cluster network bằng cách thêm `--cluster-network` vào câu lệnh bootstrap. Parameter phải là 1 subnet CIDR, ví dụ `10.90.90.0/24` hoặc `fe80::/64`.
- `cephadm bootstrap` ghi vào `/etc/ceph` những file cần thiết để truy cập vào 1 cluster mới. Central location này cho phép các gói Ceph được cài đặt trên host (ví dụ: các package có thể truy cập qua cephadm cli)
- Có thể pass bất cứ initial configuration nào vào `cephadm bootstrap` bằng cách sử dụng chúng ở dạng standard ini-style configuration file. Ví dụ: `cephadm bootstrap --config *<config-file>*` option.

```bash
cat <<EOF > initial-ceph.conf
[global]
osd crush chooseleaf type = 0
EOF
cephadm bootstrap --config initial-ceph.conf
```

- The `--ssh-user *<user>*` option cho phép chọn user để sử dụng cho SSH connection đến các host. SSH key sẽ được add vào `/home/*<user>*/.ssh/authorized_keys`. User này cần có quyền sudo và không cần password.
- Nếu sử dụng 1 container image từ 1 registry mà cần login, cần pass thêm argument `--registry-json *<path-to-json>*` với nội dung của JSON file chứa thông tin login.

```json
{
  "url": "REGISTRY_URL",
  "username": "REGISTRY_USERNAME",
  "password": "REGISTRY_PASSWORD"
}
```

Cephadm sẽ attempt login vào registry trước khi pull image và lưu thông tin login vào database. Những host khác được thêm vào cluster cũng sẽ có thể sử dụng thông tin này để authen với registry.

Cephadm không yêu cầu bất kỳ packages nào khác cài đặt trên host. Tuy nhiên ta có thể dễ dàng truy cập vào `ceph` command. 

- `cephadm shell` command chạy 1 bash shell ở trong container với tất cả các packages Ceph được cài đặt. Mặc định, file cấu hình và file keyring được tìm thấy ở `/etc/ceph` ở trên host, chúng được pass vào container nên shell sẽ có đâỳ đủ chức năng. 
  > Note: Khi thực thi trên 1 MON host, `cephadm shell` sẽ infer config từ MON container thay vì sử dụng config mặc định. Nếu cờ `--mount <path>` được pass vào, the host <path> (file hoặc directory) sẽ xuất hiện trong container tại `/mnt`.

- Để thực thi `ceph` command, sử dụng `cephadm shell -- ceph <args>`. Ví dụ: `cephadm shell -- ceph -s`. 

```bash
root@vm1:~# cephadm shell -- ceph -s
Inferring fsid 4e07ea68-b2b0-11ef-a3ca-000c298acaf4
Inferring config /var/lib/ceph/4e07ea68-b2b0-11ef-a3ca-000c298acaf4/mon.vm1/config
Using ceph image with id 'fd3234b9d664' and tag 'v19' created on 2024-09-27 21:52:09 +0000 UTC
quay.io/ceph/ceph@sha256:200087c35811bf28e8a8073b15fa86c07cce85c575f1ccd62d1d6ddbfdc6770a
  cluster:
    id:     4e07ea68-b2b0-11ef-a3ca-000c298acaf4
    health: HEALTH_WARN
            failed to probe daemons or devices
            mon vm1 is low on available space
            OSD count 0 < osd_pool_default_size 3

  services:
    mon: 1 daemons, quorum vm1 (age 2h)
    mgr: vm1.cjycvp(active, since 2h)
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

## Bước 3: Cài đặt `ceph-common` package including `ceph`, `rbd`, `mount.ceph` (for mouting CephFS file systems), etc:

```bash
cephadm add-repo --release squid
cephadm install ceph-common
```

```bash
root@vm1:~# ceph -v
ceph version 19.2.0 (16063ff2022298c9300e49a547a16ffda59baf13) squid (stable)
root@vm1:~# ceph status
  cluster:
    id:     4e07ea68-b2b0-11ef-a3ca-000c298acaf4
    health: HEALTH_WARN
            failed to probe daemons or devices
            mon vm1 is low on available space
            OSD count 0 < osd_pool_default_size 3

  services:
    mon: 1 daemons, quorum vm1 (age 2h)
    mgr: vm1.cjycvp(active, since 2h)
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

## Bước 4: Adding hosts to the cluster

Mặc định, `ceph.conf` file và 1 bản copy của `client.admin` keyring được để trong `/etc/ceph` trên tất cả các host và có label `_admin`. Label này chỉ được apply vào bootstrap host. Recommend 1 hoặc nhiều host khác được thêm vào cluster với label `_admin` để cho phép Ceph CLI (vd: `cephadm shell`) có thể dễ dàng truy cập trên nhiều host. Để add label `_label` vào các host khác, sử dụng `ceph orch host label add *<host>* _admin`.

See more: [Adding Hosts](adding-host.md)

## Bước 5: Adding MONs

Một cụm Ceph thông thường có 3 hoặc 5 Monitor daemons được phân bổ trên các host khác nhau. Recommend  deploy 5 MONs nếu có nhiều hơn 5 nodes trong cluster. 

See more: [Adding MONs](adding-mon.md)