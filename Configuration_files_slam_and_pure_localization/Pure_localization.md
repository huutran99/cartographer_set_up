Tương tự như file ntuamr_carto_slam_only_lidar.lua

### Cần tạo 1 file có dạng ntuamr_carto_pure_localization.lua

- Nội dung các parameter trong file:

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

---------------------------------------------------
TRAJECTORY_BUILDER.pure_localization_trimmer = {
  max_submaps_to_keep = 3
}
---------------------------------------------------

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

1. **max_submaps_to_keep**: Khi chạy pure_localization tối đa chỉ có 3 submap gần nhất được giữ lại để so sánh, việc này giúp tránh tình trạng giảm hiệu suất sử lý.
2. **POSE_GRAPH.constraint_builder.global_localization_min_score**: trong quá trình chạy localization, dữ liệu quét của laser sẽ tạo ra các submaps sau đó chúng so sánh với các submaps đã lưu trong quá trình vẽ bản đồ trước đó. Độ tương đồng chính là điểm số , nếu độ tương đồng lớn hơn hoặc bằng giá trị của "global_localization_min_score" thuật toán sẽ chấp nhận tư thế của robot và định vị lại.
