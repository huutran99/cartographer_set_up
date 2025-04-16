Dưới đây là file .lua slam dùng để tạo bản đồ 2D

### cần tạo 1 file có dạng filename.lua ví dụ ntuamr_carto_slam_only_lidar.lua

- nội dung các parameter trong file

```
include "map_builder.lua"
include "trajectory_builder.lua"

options = {
  map_builder = MAP_BUILDER,
  trajectory_builder = TRAJECTORY_BUILDER,
  map_frame = "map",
  tracking_frame = "base_footprint",
  published_frame = "odom", --odom
  odom_frame = "odom",
  provide_odom_frame = false, --false
  publish_frame_projected_to_2d = true,
  use_pose_extrapolator = true,
  use_odometry = false,
  use_nav_sat = false,
  use_landmarks = false,
  num_laser_scans = 1,
  num_multi_echo_laser_scans = 0,
  num_subdivisions_per_laser_scan = 1,
  num_point_clouds = 0,
  lookup_transform_timeout_sec = 0.2,
  submap_publish_period_sec = 0.3,
  pose_publish_period_sec = 5e-3,
  trajectory_publish_period_sec = 30e-3,
  rangefinder_sampling_ratio = 0.5, --affect the submap stability
  odometry_sampling_ratio = 1.,
  fixed_frame_pose_sampling_ratio = 1.,
  imu_sampling_ratio = 1.,
  landmarks_sampling_ratio = 1.,
  publish_tracked_pose = true,
}

MAP_BUILDER.use_trajectory_builder_2d = true

TRAJECTORY_BUILDER_2D.submaps.num_range_data = 15
TRAJECTORY_BUILDER_2D.min_range = 0.5  --0.5
TRAJECTORY_BUILDER_2D.max_range = 25.	 --25.
TRAJECTORY_BUILDER_2D.missing_data_ray_length = 25.5
TRAJECTORY_BUILDER_2D.use_imu_data = false 
TRAJECTORY_BUILDER_2D.use_online_correlative_scan_matching = true
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.linear_search_window = 0.1  --0.1
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.translation_delta_cost_weight = 10
TRAJECTORY_BUILDER_2D.real_time_correlative_scan_matcher.rotation_delta_cost_weight = 1e-1
--TRAJECTORY_BUILDER_2D.motion_filter.max_angle_radians=math.rad(0.2)

-- New config
POSE_GRAPH.constraint_builder.global_localization_min_score=0.62

POSE_GRAPH.optimization_problem.huber_scale = 1e2
POSE_GRAPH.optimize_every_n_nodes = 2 
POSE_GRAPH.constraint_builder.min_score = 0.62

return options
```

### Một số param cần thay đổi phù hợp với robot

1. **map_frame**: ID frame trong ROS được sử dụng để xuất bản các bản đồ con (submaps), là parent frame của các frame khác như odom, base_footprint, base_link,... Thường được cấu hình là "map".
2. **tracking_frame**: ID frame được theo dõi bởi thuật toán SLAM. Nếu sử dụng IMU nó phải ở đúng vị trí của nó, mặc dù nó có thể bị xoay. Thường được cấu hình là "imu_link".
3. **published_fram**: ID frame được sử dụng làm child frame để xuất bản tư thế robot. Cấu hình là "odom" nếu "odom" khung odom được cung cấp bởi 1 bộ phận khác. Nếu không thường sẽ được đặt là "base_link".
4. **odom_frame**: chỉ được sử dụng nếu "**provide_odom_frame** được đặt là true. Khung giữa **published_frame** và **map_frame** dùng để xuất bản kết quả SLAM. Thường được cấu hình là "odom".
5. **provide_odom_frame**: nếu được đặt là true sẽ liên tục được xuất bản odom_frame trong map_frame.
6. **publish_frame_projected_to_2d**: nếu được đặt là true, các tư thế sẽ chỉ xuất bản dưới dạng tọa độ 2D.
7. **use_odometry**; nếu được đặt là true, dữ liệu odom sẽ đưa vào sử dụng trong SLAM.
8. **num_laser_scans**: số lượng topic laser được sử dụng, topic thường được đặt là "scan" nếu 1 laser được sử dụng và "scan_1", "scan_2",v.v nếu nhiều laser được sử dụng.
9. **TRAJECTORY_BUILDER_2D.submaps.num_range_data**: số lượt quét cần thiết để tạo ra 1 submap.
10. **TRAJECTORY_BUILDER_2D.min_range**: phạm vi quét tối thiểu của laser được xem là hợp lệ để ghi nhận.
11. **TRAJECTORY_BUILDER_2D.max_range**: phạm vi tối đa của laser được xem là hợp lệ để ghi nhận.
12. **TRAJECTORY_BUILDER_2D.use_imu_data**: nếu được đặt là true, dữ liệu từ imu sẽ được đưa vào SLAM.
13. **POSE_GRAPH.constraint_builder.min_score**: submap sẽ được giữ lại nếu điểm số của submap bằng hoặc lớn hơn điểm số được set(giá trị từ 0 đến 1).

