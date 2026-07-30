### GIAI ĐOẠN 1: TẠO KHÔNG GIAN LÀM VIỆC VÀ TẢI MÃ NGUỒN

Trước khi mời một robot về nhà, chúng ta phải xây cho nó một căn nhà (Workspace) gọn gàng và chuẩn cấu trúc.

**1. Tạo thư mục Workspace**
Mở Terminal lên và chạy 2 lệnh sau để tạo một thư mục chuyên dụng chứa code cho cánh tay KUKA:

```bash
mkdir -p ~/kuka_ws/src
cd ~/kuka_ws/src

```

**2. Tải mã nguồn (Clone) từ GitHub**
Khi đang đứng trong thư mục `src`, bạn dùng lệnh `git clone` để tải toàn bộ thiết kế 3D (URDF/XACRO) của cánh tay KUKA về máy.

```bash
git clone <đường-link-github-kuka-của-bạn>

```

*(Thay `<đường-link-github-kuka-của-bạn>` bằng link Repository chứa KUKA URDF mà bạn muốn dùng).*

---

### GIAI ĐOẠN 2: BIÊN DỊCH VÀ NẠP MÔI TRƯỜNG

Phần mềm cần được biên dịch để ROS 2 có thể hiểu và kết nối các tệp tin lại với nhau.

**1. Cài đặt các thư viện phụ thuộc (Dependencies)**
Để chắc chắn máy tính không bị thiếu thư viện khi chạy, hãy dùng lệnh sau (nhớ chạy từ thư mục gốc `kuka_ws`):

```bash
cd ~/kuka_ws
rosdep install --from-paths src --ignore-src -r -y

```

**2. Biên dịch hệ thống (Build)**
Dùng công cụ `colcon` tiêu chuẩn của ROS 2 để đóng gói mã nguồn:

```bash
colcon build --symlink-install

```

**3. Nạp môi trường (Source)**
Đây là bước cực kỳ quan trọng! Máy tính cần được thông báo rằng ROS 2 Jazzy đang hoạt động và Workspace của bạn đã sẵn sàng. Bạn chạy 2 lệnh sau:

```bash
source /opt/ros/jazzy/setup.bash
source install/setup.bash

```

---

### GIAI ĐOẠN 3: "BƠM LINH HỒN" VỚI MOVEIT SETUP ASSISTANT

Bây giờ là lúc chúng ta dạy cho cánh tay KUKA biết nó có bao nhiêu khớp, giới hạn xoay là bao nhiêu và làm sao để không tự đập vào thân nó.

**1. Khởi động công cụ**
Chạy lệnh sau trên Terminal:

```bash
ros2 run moveit_setup_assistant moveit_setup_assistant

```

**2. Thao tác trên giao diện đồ họa (GUI)**
Một cửa sổ mới sẽ hiện ra. Hãy làm theo đúng thứ tự các tab ở cột bên trái:

* **Start:** Chọn *Create New MoveIt Configuration Package*. Bấm nút *Browse* và tìm đến file định dạng `.urdf` hoặc `.xacro` của robot KUKA trong thư mục `src` của bạn. Sau đó bấm *Load Files*.
* **Self-Collisions:** Chuyển sang tab này, kéo thanh trượt *Sampling Density* lên mức High, rồi bấm **Generate Collision Matrix**. Phần mềm sẽ tự mô phỏng hàng vạn va chạm để biết cách né tránh.
* **Virtual Joints:** Bấm *Add Virtual Joint*. Đặt tên là `virtual_joint`, kiểu kết nối chọn `fixed`, Parent Frame ghi là `world` và Child Link chọn tên cái đế của robot (thường là `base_link`).
* **Planning Groups:** Đây là nơi tạo nhóm điều khiển. Bấm *Add Group*. Đặt tên nhóm là `kuka_arm`. Chọn thuật toán giải động học là `kdl_kinematics_plugin/KDLKinematicsPlugin`. Sau đó bấm *Add Kinematic Chain* và chọn từ khớp nối cơ sở (Base) đến điểm chót (End Effector - TCP).
* **Robot Poses:** Bấm *Add Pose*. Kéo các thanh trượt để cánh tay tạo thành tư thế nghỉ (gập gọn lại) và đặt tên cho tư thế đó là `home`.
* **ROS 2 Controllers:** Bấm nút **Auto Add...** để phần mềm tự động sinh ra mã điều khiển động cơ cho các khớp.

> **🛑 BƯỚC QUAN TRỌNG NHẤT: LƯU FILE (CONFIGURATION FILES)**
> Tại tab cuối cùng, phần mềm sẽ yêu cầu bạn tạo một thư mục mới để xuất toàn bộ cấu trúc thần kinh này ra.
> **Quy tắc BẮT BUỘC:** Tên của gói (package) này **phải kết thúc bằng cụm từ `_config**`.
> Ví dụ, nếu cánh tay của bạn tên là *kuka_kr6*, bạn phải đặt tên package này là: **`kuka_kr6_config`** (hoặc `kuka_moveit_config`).
> Việc thêm đuôi `_config` này là một tiêu chuẩn ngầm định vô cùng nghiêm ngặt của ROS 2. Nếu thiếu nó, các file khởi chạy sau này sẽ không thể tìm thấy nhau, hệ thống báo lỗi không nhận diện được đường đi và toàn bộ cấu trúc sẽ sụp đổ.
> Sau khi đặt tên chuẩn xác, hãy bấm **Generate Package** để hoàn tất.

---

### GIAI ĐOẠN 4: TRÌNH DIỄN TRÊN RVIZ2

Công đoạn khó nhất đã qua. Bây giờ hãy tận hưởng thành quả của mình.

**1. Biên dịch lại Workspace**
Vì bạn vừa sinh ra một package `_config` mới toanh từ Setup Assistant, bạn phải biên dịch lại thư mục làm việc để ROS 2 nhận diện nó.
Đóng cửa sổ Setup Assistant, quay lại Terminal và chạy:

```bash
cd ~/kuka_ws
colcon build
source install/setup.bash

```

**2. Khởi chạy hiển thị trên RViz2**
MoveIt 2 đã tự động tạo sẵn cho bạn một file khởi chạy bản demo cực kỳ xịn sò. Chỉ cần gõ lệnh sau (nhớ thay tên package bằng cái tên có chữ `_config` bạn vừa đặt):

```bash
ros2 launch kuka_moveit_config demo.launch.py

```

**3. Thao tác điều khiển trên RViz2**

* Cửa sổ RViz2 mở ra, bạn sẽ thấy cánh tay KUKA 3D sừng sững ở giữa màn hình.
* Ở điểm chót của cánh tay sẽ có một **quả cầu màu xanh dương/đỏ** kèm theo các vòng tròn tọa độ.
* Hãy dùng chuột **kéo quả cầu đó** đến một vị trí bất kỳ trong không gian (đây là điểm đến mong muốn của bạn).
* Nhìn sang bảng điều khiển *MotionPlanning* ở cột bên trái, bấm vào nút **Plan**. Bạn sẽ thấy một vệt mờ hiển thị quỹ đạo di chuyển dự kiến.
* Nếu quỹ đạo đẹp và không bị va chạm, hãy bấm nút **Execute**.

Cánh tay KUKA của bạn sẽ lướt đi vô cùng mượt mà đến đúng tọa độ bạn chỉ định!
