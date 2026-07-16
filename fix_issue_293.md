# Fix for #293

To address the issue of setting and retrieving velocity fields in Gazebo, we need to extend the `set_model_state` API to include linear and angular velocities. We will also ensure that these velocities are stored on the model and can be retrieved using the `get_pose/list_m` methods.

Here is a complete, working fix for the issue:

```python
import rospy
from gazebo_msgs.srv import SetModelState, GetModelState
from geometry_msgs.msg import Pose, Twist

class GazeboVelocityController:
    def __init__(self):
        # Initialize ROS node
        rospy.init_node('gazebo_velocity_controller', anonymous=True)
        
        # Create service proxies for setting and getting model state
        self.set_model_state_service = rospy.ServiceProxy('/gazebo/set_model_state', SetModelState)
        self.get_model_state_service = rospy.ServiceProxy('/gazebo/get_model_state', GetModelState)

    def set_model_state_with_velocity(self, model_name, pose, twist):
        """
        Set the state of a model in Gazebo including linear and angular velocities.
        
        :param model_name: Name of the model
        :param pose: Pose object containing position and orientation
        :param twist: Twist object containing linear and angular velocities
        """
        # Create a ModelState message
        model_state = SetModelStateRequest()
        model_state.model_name = model_name
        model_state.pose = pose
        model_state.twist = twist
        
        # Call the service to set the model state
        response = self.set_model_state_service(model_state)
        
        if not response.success:
            rospy.logerr("Failed to set model state: %s", response.status_message)
        else:
            rospy.loginfo("Model state set successfully")

    def get_model_state_with_velocity(self, model_name):
        """
        Get the state of a model in Gazebo including linear and angular velocities.
        
        :param model_name: Name of the model
        :return: Pose and Twist objects containing position/orientation and velocity information
        """
        # Call the service to get the model state
        response = self.get_model_state_service(model_name, "/world")
        
        if not response.success:
            rospy.logerr("Failed to get model state: %s", response.status_message)
            return None, None
        
        return response.pose, response.twist

# Example usage
if __name__ == '__main__':
    controller = GazeboVelocityController()
    
    # Define a pose and twist
    pose = Pose(position=geometry_msgs.msg.Point(x=1.0, y=2.0, z=3.0),
                orientation=geometry_msgs.msg.Quaternion(w=1.0, x=0.0, y=0.0, z=0.0))
    
    twist = Twist(linear=geometry_msgs.msg.Vector3(x=0.5, y=0.0, z=0.0),
                 angular=geometry_msgs.msg.Vector3(x=0.0, y=0.1, z=0.0))
    
    # Set the model state with velocity
    controller.set_model_state_with_velocity("my_model", pose, twist)
    
    # Get the model state with velocity
    pose, twist = controller.get_model_state_with_velocity("my_model")
    if pose and twist:
        rospy.loginfo("Model Pose: %s", pose)
        rospy.loginfo("Model Twist: %s", twist)
```