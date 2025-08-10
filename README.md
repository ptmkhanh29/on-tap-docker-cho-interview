# on-tap-docker-cho-interview
Nơi mình lưu lại phần ôn tập của mình về Docker để chuẩn bị interview

## 5 Điểm Tương Đồng Giữa Docker và Máy Ảo (VM)

| Đặc Điểm | Máy Ảo (VM) | Docker Container |
|----------|------------|-----------------|
| **Cách ly tiến trình** | Sử dụng hypervisor để cách ly hoàn toàn tiến trình giữa các VM | Sử dụng Linux namespaces để cách ly tiến trình giữa các container |
| **Hệ thống tập tin** | Chứa đầy đủ hệ điều hành độc lập (kernel, thư viện, ứng dụng hệ thống) với dung lượng lớn (GB) | Chỉ chứa ứng dụng và dependencies thiết yếu (không có kernel) với dung lượng nhỏ (MB), chia sẻ kernel với host |
| **Mạng ảo** | Sử dụng virtual NIC (Network Interface Card) giả lập phần cứng thật với MAC/IP riêng | Sử dụng virtual network interface (veth pair) kết nối qua Linux bridge (docker0) với IP riêng |
| **Khởi chạy từ image** | Khởi chạy từ disk image chứa toàn bộ hệ thống (.vmdk, .qcow2) | Khởi chạy từ Docker image được xây dựng từ các layer read-only |
| **Tương thích hệ điều hành** | Có thể chạy guest OS khác biệt hoàn toàn với host OS (VD: Windows trên Linux) | Chỉ chạy được container cùng loại kernel với host (VD: Linux container trên Linux host) |

## 7 điểm khác nhau giữa Docker và Virtual Machine (VM)

![Image](./00_images/image_00.avif)

| Tiêu chí                | Docker Container                                                                 | Virtual Machine (VM)                                                              |
|-------------------------|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Kiến trúc**           | Chạy trực tiếp trên **Host OS** thông qua **Docker Engine**.                     | Chạy trên **Hypervisor** (VD: VMware, VirtualBox) và mỗi VM có **Guest OS** riêng.|
| **Hệ điều hành**        | Chia sẻ kernel của **Host OS**, không cần cài đặt OS riêng cho từng container.    | Mỗi VM chạy **một Guest OS đầy đủ** độc lập, tách biệt với Host OS.               |
| **Tài nguyên**          | Nhẹ hơn vì không cần OS riêng; chỉ chứa app + thư viện cần thiết.                 | Nặng hơn do mỗi VM cần RAM, CPU, và ổ đĩa cho cả hệ điều hành.                    |
| **Khởi tạo**            | Khởi chạy rất nhanh (vài giây) vì chỉ start process trong Host OS.                | Khởi chạy lâu hơn (vài chục giây - phút) do phải boot toàn bộ OS.                  |
| **Hiệu suất**           | Gần như hiệu năng native vì không qua lớp ảo hóa toàn bộ phần cứng.                | Chậm hơn do overhead từ Hypervisor và OS riêng.                                   |
| **Mức độ cô lập**       | Cô lập ở mức tiến trình + không gian tên (namespace), bảo mật ở mức OS.           | Cô lập hoàn toàn phần cứng ảo, bảo mật cao hơn nhưng tốn nhiều tài nguyên.        |
| **Tính di động**        | Image nhẹ, dễ build và deploy trên nhiều môi trường giống nhau.                   | Di chuyển VM khó hơn, file ảnh (VM image) thường lớn hơn nhiều lần container image.|

## So sánh Container Networking vs VM Networking

| Đặc điểm               | Docker Container Networking                          | Virtual Machine Networking                          |
|------------------------|-----------------------------------------------------|----------------------------------------------------|
| **Cách ly mạng**       | Sử dụng Linux network namespace (như "phòng riêng" trong cùng căn nhà) | Mỗi VM có virtual NIC độc lập (như nhà riêng biệt hoàn toàn) |
| **Cấp phát IP**        | IP ảo trong docker bridge network (như số phòng nội bộ) | IP riêng cho từng VM (như địa chỉ nhà riêng)       |
| **Triển khai dịch vụ** | 1 service chạy trên nhiều container (như đội chuyên gia) | Nhiều service chạy chung 1 VM (như phòng ban đa năng) |
| **Cân bằng tải**       | Tích hợp sẵn trong Docker/K8s (tự động phân chia công việc) | Cần cài đặt riêng (Nginx/HAProxy như thuê quản lý bên ngoài) |
| **Hiệu suất**          | Gần native qua veth pairs (như giao tiếp trực tiếp)   | Overhead do ảo hóa phần cứng (như qua trung gian)   |
| **Khả năng mở rộng**   | Hỗ trợ hàng trăm container/host (như xe máy dễ di chuyển) | Thường <20 VM/host (như xe tải chiếm diện tích)     |
| **Công nghệ**          | Docker0 bridge + iptables (hệ thống nhẹ)            | OVS/bridge + hypervisor (hệ thống phức tạp)        |

## Ví dụ minh họa:

1. **Network Isolation**:
   ```bash
   # Container: Tạo network namespace
   docker run --net=my_bridge -it alpine sh
   (Giống như tạo phòng riêng trong nhà chung)

   # VM: Tạo virtual NIC
   virsh attach-interface vm1 bridge br0
   (Giống như lắp đường dây mạng riêng cho nhà mới)