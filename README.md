# lidar_align

## A simple method for finding the extrinsic calibration between a 3D lidar and a 6-dof pose sensor
The method makes use of the property that pointclouds from lidars appear more 'crisp' when the calibration is correct. The system attempts to find the transform between a lidar and the given poses such that the distance between any two lidar points in the resulting fused pointcloud is minimized. Poses can be provided as a TransformStamped message or in Maplab's CSV format

This library is pretty new and only tested on a couple of datasets, so expect some issues when you first run it.

## Estimation proceedure
The estimation is usually performed in two stages.

* First set `local` to `false` and `range` to encompass all sane values for example `[0.5, 0.5, 0.5, 3.2, 3.2, 3.2, 0.1]` will explore all rotations, plus all translations within 0.5m. The result of this optimization will get you in the ballpark but will most likely have significant error.
* Secondly take the solution from above and use this to set the `inital_guess`, then rerun with `local` set to true and a smaller range for example `[0.1, 0.1, 0.1, 0.5, 0.5, 0.5, 0.01]`.

## Deps
This package depends on ROS, PCL and ncurses. Ncurses can be installed with `sudo apt install libncurses5-dev`

## Parameters
------

### Scan Parameters
| Parameter | Description | Default |
| --------------------  |:-----------:| :-------:|
| `min_point_distance` |  Minimum range a point can be from the lidar and still be included in the optimization. | 0.0 |
| `max_point_distance` |  Maximum range a point can be from the lidar and still be included in the optimization. | 100.0 |
| `keep_points_ratio` |  Ratio of points to use in the optimization (runtimes increase drastically as this is increased). | 0.01 |
| `min_return_intensity` | The minimum return intensity a point requires to be considered valid. | -1.0 |
| `motion_compensation` |  If the movement of the lidar during a scan should be compensated for. | true |
| `estimate_point_times` | Uses the angle of the points in combination with `lidar_rpm` and `clockwise_lidar` to estimate the time a point was taken at. | false |
| `lidar_rpm` | Spin rate of the lidar in rpm, only used with `estimate_point_times`. | 600 |
| `clockwise_lidar` | True if the lidar spins clockwise, false for anti-clockwise, only used with `estimate_point_times`. | false |

### IO Parameters
| Parameter | Description | Default |
| --------------------  |:-----------:| :-------:|
| `use_n_scans` |  Optimization will only be run on the first n scans of the dataset. | 2147483647 |
| `input_bag_path` |  Path of rosbag containing sensor_msgs::PointCloud2 messages from the lidar. | N/A  |
| `transforms_from_csv` | True to load scans from a csv file, false to load from the rosbag. | false |
| `input_csv_path` |  Path of csv generated by Maplab, giving poses of the system to calibrate to. | N/A |
| `output_pointcloud_path` |  If set, a fused pointcloud will be saved to this path as a ply when the calibartion finishes. | "" |
| `output_calibration_path` |  If set, a text document giving the final transform will be saved to this path when the calibration finishes. | "" |

### Alinger Parameters
| Parameter | Description | Default |
| --------------------  |:-----------:| :-------:|
| `local` |  True for local optimization, false for a much slower global optimization | true |
| `inital_guess` |  Inital guess to the calibration (x, y, z, rotation vector, time offset) | [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] |
| `range` |  Range around the inital guess the optimizer will search for a solution in | [1.0, 1.0, 1.0, 3.15, 3.15, 3.15, 0.0] |
| `max_evals` | Maximum number of function evaluations to run | 1000 |
| `xtol` | Tolerance of final solution | 0.000001 |
| `knn_batch_size` | Number of points to send to each thread when finding nearest points | 1000 |
| `knn_k` | Number of neighbours to consider in error function | 1 |
| `knn_max_dist` | Error between points is limited to this value | 0.1 |
| `time_cal` | True to perform time offset calibration | false |
