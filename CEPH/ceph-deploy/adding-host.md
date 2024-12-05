# Ceph Host Management

## Listing hosts

```bash
ceph orch host ls [--format yaml] [--host-pattern <name>] [--label <label>] [--host-status <status>] [--detail]
```

Ở trong command trên, arguments "host-pattern", "label", "host-status" là optional, được sử dụng cho việc filering

- `host-pattern`: regex pattern để filter host names và trả về các host names match với pattern
- `label`: filter host names với label cụ thể
- `host-status`: filter host names với status cụ thể (vd: `offline`, `maintenance`)
- Bất kỳ sự kết hợp của việc filtering flags trên đều hợp lệ. Nó có thể là filter theo name, label và status cùng một lúc hoặc chỉ filter theo bất kỳ subset nào của chúng.
- `--detail`: param cung cấp thêm thông tin về host, bao gồm cả label và status.

```bash
ceph orch host ls --detail
```

## Adding hosts

### Bước 1: Install public SSH key của cluster ở host mới vào root user authorized_keys file của host đó

```bash
ssh-copy-id -f -i /etc/ceph/ceph.pub root@<new-host>
```

ví dụ:

```bash
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph-host-2
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph-host-3
```

### Bước 2: Add host vào cluster

```bash
ceph orch host add *<new-host>* [*<ip>*] [*<label1>...*]
```

ví dụ:

```bash
ceph orch host add ceph-host-2 192.168.103.148
ceph orch host add ceph-host-3 192.168.103.149
```

Nếu 1 địa chỉ không được cung cấp, host name sẽ ngay lập tức được resolve qua DNS và kết quả sẽ được sử dụng.

Một hoặc nhiều label có thể được thêm lập tức vào host mới, mặc định là `_admin` label sẽ làm cho cephadm giữ 1 bản copy của `ceph.conf` file và 1 `client.admin` keyring file trên `/etc/ceph`

```bash
ceph orch host add ceph-host-4 192.168.103.151 --label _admin
```

## Removing hosts

1 Host có thể được remove từ cluster sau khi tất cả daemons được remove từ host đó.

```bash
ceph orch host drain *<host>*
```

Nhãn `_no_schedule` và `_no_conf_keyring` sẽ được thêm vào host, ngăn chặn việc schedule thêm daemon và cập nhật cấu hình và keyring.

See more: [Special host labels](host-labels.md)

Nếu muốn drain daemons nhưng vẫn để `ceph.conf` và keyring file trên host, có thể sử dụng `--keep-conf-keyring` flag

```bash
ceph orch host drain *<host>* --keep-conf-keyring
```

Điều này sẽ apply `_no_schedule` label vào host nhưng không apply `_no_conf_keyring` label.

Tất cả OSDs trên host sẽ được scheduled để removed. Check status remove của OSDs:

```bash
ceph orch osd rm status
```

See more: [Removing OSDs](osd-management.md)