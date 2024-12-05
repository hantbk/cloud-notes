# Special host labels

- `_no_schedule`: Không schedule thêm daemon trên host
Nhãn này ngăn chặn cephadm schedule thêm daemon trên host. Nếu nó được thêm vào host đang chạy daemon, daemon sẽ không bị remove, nhưng daemon mới sẽ không được schedule thêm. (ngoại trừ OSDs, sẽ không được remove tự động)

- `_no_conf_keyring`: Không cập nhật cấu hình hoặc keyring trên host
  Nhãn này giống như `_no_schedule`, nhưng thay vì làm việc với daemons nó hoạt động với client keyring và config file được quản lý bởi cephadm

- `_no_autotune_memory`: Không tự động tune memory

Nhãn ngăn chặn daemon memory từ việc tự động tune kể cả khi `osd_memory_target_auto_tune` hoặc options tương tự được bật cho 1 hoặc nhiều daemons trên host đó.

- `_admin`: Cung cấp ceph.conf và client.admin keyring
  Mặc định, nhãn `_admin` được apply vào host đầu tiên ở trong cluster (nơi bootstrap được chạy) và `client.admin` key được cung cấp cho host đó thông qua `ceph orch client-keyring ...`. Việc thêm nhãn đến các host khác sẽ làm cephadm deploy config và keyring file cho host đó ở `/etc/ceph/`.