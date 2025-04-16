Hướng dẫn cài đặt Cartographer trên ROS Noetic hệ điều hành ubuntu 20.04.6

Để build Cartographer ROS, nên sử dụng wstool và rosdep. Để quá trình build diễn ra nhanh hơn nên sử dụng Ninja.
### Các bước setup và install cartographer

- Trên ROS Noetic sử dụng các câu lệnh sau để cài đặt công cụ wstool và ninja.

```
sudo apt-get update
sudo apt-get install -y python3-wstool python3-rosdep ninja-build stow
```

- Sau khi các công cụ được cài đặt, tạo không gian làm việc cartographer_ws

```
mkdir cartographer_ws
cd cartographer_ws
wstool init src
wstool merge -t src https://raw.githubusercontent.com/cartographer-project/cartographer_ros/master/cartographer_ros.rosinstall
wstool update -t src
```

Bây giờ cần cài đặt các dependence của cartographer_ros. Đầu tiên chúng ta cần sử dụng rosdep để cài đặt các gói cần thiết. Lệnh 'sudo rosdep init' sẽ in ra lỗi nếu bạn đã thực hiện lệnh này kể từ khi cài đặt ROS. Lỗi này có thể bỏ qua.

- Cài đặt dependency bằng rosdep

```
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro=${ROS_DISTRO} -y
```

- Cài đặt thư viện abseil-cpp

```
src/cartographer/scripts/install_abseil.sh
```

- Cần gỡ cài đặt các phiên bản khác của abseil-cpp để tránh xung đột

```
sudo apt-get remove ros-${ROS_DISTRO}-abseil-cpp
```

- Build and install

```
catkin_make_isolated --install --use-ninja
```




