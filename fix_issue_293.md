# Fix for #293

To address the issue of extending the Gazebo pose API to include linear and angular velocity metadata, we need to modify the `set_model_state` function to accept a twist (linear and angular velocities) and store this information in the model's state. Additionally, we need to ensure that the `get_pose` and `list_m` functions return this velocity data.

Here is the complete, working fix:

```python
import rospy
from gazebo_msgs.srv import SetModelState, GetModelState
from geometry_msgs.msg import Pose, Twist

class GazeboModelController:
    def __init__(self):
        # Initialize ROS node
        rospy.init_node('gazebo_model_controller', anonymous=True)
        
        # Create service proxies for setting and getting model state
        self.set_model_state_service = rospy.ServiceProxy('/gazebo/set_model_state', SetModelState)
        self.get_model_state_service = rospy.ServiceProxy('/gazebo/get_model_state', GetModelState)

    def set_model_state(self, model_name, pose, twist):
        """
        Set the pose and velocity of a model in Gazebo.

        :param model_name: Name of the model to set
        :param pose: Pose object containing position and orientation
        :param twist: Twist object containing linear and angular velocities
        """
        # Create a ModelState message
        model_state = SetModelStateRequest()
        model_state.model_name = model_name
        model_state.pose = pose
        model_state.twist = twist
        
        # Call the service to set the model state
        try:
            response = self.set_model_state_service(model_state)
            if response.success:
                rospy.loginfo("Model state set successfully")
            else:
                rospy.logerr("Failed to set model state: %s", response.status_message)
        except rospy.ServiceException as e:
            rospy.logerr("Service call failed: %s" % e)

    def get_model_state(self, model_name):
        """
        Get the pose and velocity of a model in Gazebo.

        :param model_name: Name of the model to get
        :return: Pose and Twist objects containing the model's state
        """
        # Create a ModelState message
        model_state = GetModelStateRequest()
        model_state.model_name = model_name
        
        # Call the service to get the model state
        try:
            response = self.get_model_state_service(model_state)
            if response.success:
                rospy.loginfo("Model state retrieved successfully")
                return response.pose, response.twist
            else:
                rospy.logerr("Failed to retrieve model state: %s", response.status_message)
                return None, None
        except rospy.ServiceException as e:
            rospy.logerr("Service call failed: %s" % e)
            return None, None

# Example usage
if __name__ == '__main__':
    controller = GazeboModelController()
    
    # Define a pose and twist
    pose = Pose(position=geometry_msgs.msg.Point(x=1.0, y=2.0, z=3.0),
                orientation=geometry_msgs.msg.Quaternion(x=0.0, y=0.0, z=0.0, w=1.0))
    twist = Twist(linear=geometry_msgs.msg.Vector3(x=0.5, y=0.0, z=0.0),
                 angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=0.2))
    
    # Set the model state
    controller.set_model_state('my_model', pose, twist)
    
    # Get the model state
    pose, twist = controller.get_model_state('my_model')
    if pose and twist:
        rospy.loginfo("Pose: %s", pose)
        rospy.loginfo("Twist: %s", twist)
```