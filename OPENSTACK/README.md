# OpenStack

Định nghĩa về Openstack “Open source software for building private and public clouds”, tức OpenStack là một phần mềm mã nguồn mở, dùng để triển khai Cloud Computing, bao gồm Private Cloud và Public cloud (nhiều tài liệu giới thiệu là Cloud Operating System). Tên các phiên bản Openstack được bắt đầu theo thứ tự A, B, C, D … trong bảng chữ cái.

# Các thành phần cơ bản

## OpenStack Identity Server (code-name Keystone)

Keystone là module chịu trách nhiệm đảm bảo việc bảo mật, kiểm soát truy cập tới tất cả các tài nguyên trên hệ thống Openstack.

Các tính năng chính:

- Cung cấp dịch vụ xác thực trên Cloud
- Hỗ trợ nhiều kiểu xác thực
- Phân quyền theo Role-base Access Control (RBAC)

See more: [Keystone](./Keystone/)

## OpenStack Compute (code-name Nova)
Nova là module quản lý và cung cấp máy ảo (VM). Nova hỗ trợ nhiều công nghệ ảo hóa khác nhau, bao gồm KVM, QEMU, LXC, XenServer… Bản thân Nova không chứa các phần mêm ảo hóa, thay vào đó nó sẽ chưa các Driver tương tác, điều khiển các kỹ thuật ảo hóa (Công nghệ ảo hóa)

Các tính năng chính:

- Thành phần quản lý máy ảo (`Virtual Compute Instance`)
- Cung cấp API quản trị (`Nova API hay Openstack Nova API`)
- Hỗ trợ nhiều công nghệ ảo hóa: `Xen, KVM, QEMU, vSphere, Hyper-V`

See more: [Nova](./Nova/)

