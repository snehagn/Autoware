#1878, #1305 (https://github.com/autowarefoundation/autoware/pull/<****>)
init_pos_gnss (int 32, why?)
local transform (?)

Data:

struct pose
{
  double x;
  double y;
  double z;
  double roll;
  double pitch;
  double yaw;
};

static pose predict_pose_gnss, current_pose_gnss
static pose predict_pose_imu_gnss, current_pose_imu_gnss
static pose predict_pose_gnss_odom, current_pose, gnss_odom


Added publishers

Functions:
gnss_calc()
imu_gnss_calc()
odom_gnss_calc()


set gnss -> velocity if available (only imu velocity is currently available)
we have current_gnss_pose -> I am adding current_pose_gnss, etc. Use gnss_callback for init_pose and integrate
merge gnss_update_callback into gnss_callback by reference.


localizer pose, current pose and previous pose - without gnss/imu/odom -> all 0s
   diff_x = current_pose.x - previous_pose.x;
    diff_y = current_pose.y - previous_pose.y;
    diff_z = current_pose.z - previous_pose.z;
    diff_yaw = current_pose.yaw - previous_pose.yaw;
    diff = sqrt(diff_x * diff_x + diff_y * diff_y + diff_z * diff_z);
    
    
    pose trans_current_pose = convertPoseIntoRelativeCoordinate(current_pose, previous_pose)
    
      if (_get_height == true && map_loaded == 1)
  {
    double min_distance = DBL_MAX;
    double nearest_z = current_pose.z;
    for (const auto& p : map)
    {
      double distance = hypot(current_pose.x - p.x, current_pose.y - p.y);
      if (distance < min_distance)
      {
        min_distance = distance;
        nearest_z = p.z;
      }
    }
    current_pose.z = nearest_z;
  }
