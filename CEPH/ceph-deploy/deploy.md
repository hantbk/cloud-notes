# Triển khai cụm Ceph với cephadm

We look into using the cephadm tooling to bootstrap and configure a small cluster with 3 drives and multiple hosts. We go into how cephadm administers different resources and shares them between hosts.

## Bước 1: Installing main host

Đầu tiên ta cần `curl` để fetch cephadm application

```bash
sudo apt install -y curl
```

Sau đó ta download application và make it executable

```bash
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
chmod +x cephadm
```

Cephadm có thể sử dụng để thiết lập phiên bản muốn install, chọn Pacific để cài đặt cephadm

```bash
./cephadm add-repo --release pacific
./cephadm install
```

Bootstraping a cluster
