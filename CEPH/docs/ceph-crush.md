# CRUSH: Controlled, Scalable, Decentralized Placement of Replicated Data

Các Ceph Client và Ceph OSD Daemon đều sử dụng thuật toán CRUSH để tính toán thông tin về vị trí đối tượng thay vì dựa vào một bảng tra cứu tập trung (central lookup table). CRUSH cung cấp một cơ chế quản lý dữ liệu tốt hơn so với các phương pháp cũ, và cho phép mở rộng quy mô lớn bằng cách phân phối công việc cho tất cả các OSD Daemon trong cụm và các máy khách giao tiếp với chúng. CRUSH sử dụng sao chép dữ liệu thông minh để đảm bảo khả năng chịu lỗi, điều này phù hợp hơn với lưu trữ ở quy mô siêu lớn.

