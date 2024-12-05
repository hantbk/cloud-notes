# Remove an OSD

## Step 1: Di chuyển tất cả PGs ra khỏi OSD
## Step 2: Remove PG-tree OSD từ cluster

```bash
ceph orch osd rm *<osd-id(s)>* [--replace] [--force]
```

ví dụ:

```bash
ceph orch osd rm 0 
```