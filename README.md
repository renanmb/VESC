# How to get VESC working with ROS

originally from: https://github.com/mrsd16teamd/loco_car/wiki/VESC


ROS drivers for the VESC are provided by the MIT RACECAR team:

https://github.com/mit-racecar/vesc

After cloning, you might need to install the ros serial package for it to compile.

sudo apt-get install ros-indigo-serial

Once installed, catkin_make

The vesc_driver package establishes a connection to the VESC via UART.

Be sure to check the default port in the launch file: roscd vesc_driver/launch

Edit the vesc_driver_node.launch file. Once done, do roslaunch vesc_driver vesc_driver_node.launch

Now, do a rostopic list and you will be able to see several topics be published/listened on
<ul>
  <li>/commands/motor/brake</li>
  <li>/commands/motor/current</li>
  <li>/commands/motor/duty_cycle</li>
  <li>/commands/motor/position</li>
  <li>/commands/motor/speed</li>
  <li>/commands/servo/position</li>
  <li>/sensors/core</li>
  <li>/sensors/servo_position_command</li>
</ul>  
Do a rostopic echo /sensors/core to view certain sensor information like:

state:
voltage_input: 12.0
temperature_pcb: 28.7
current_motor: -0.06
current_input: 0.0
speed: -193.0
duty_cycle: -0.014
charge_drawn: 18.0
charge_regen: 0.0
energy_drawn: 223.0
energy_regen: 2.0
displacement: -2077.0
distance_traveled: 16301.0
fault_code: 0
Next, we can publish on the /commands/motor/speed topic to execute RPM commands. From the CLI, you can do:

rostopic pub -r 20 /commands/motor/speed -- std_msgs/Float64 -200

The syntax is rostopic pub --args <topic> <msg type> <value>

In this case, the -r 20 argument makes it publish at a constant rate of 20Hz, the -- after the topic is needed to specify that there are no further arguments so that it can parse negative values correctly.

This will cause the wheel to spin at -200 RPM, given that the VESC motor params have be set correctly with the BLDC_Tool.

Servo command:

rostopic pub -r 20 /commands/servo/position -- std_msgs/Float64 45

servo command value range is still unknown...

Getting odometry from the VESC

roslaunch vesc_ackermann vesc_to_odom_node.launch

Launch will fail, as we need to specify 5 rosparams:

speed_to_erpm_gain
speed_to_erpm_offset
steering_angle_to_servo_gain
steering_angle_to_servo_offset
wheelbase
These can be set manually via CLI by running:
rosparam set speed_to_erpm_gain -1664

erpm gain of -1664 is what the VESC + MST is currently tuned for. offset should be 0, may go back to investigate whether we can use this to account for steady-state error

TODO: set the rosparams from the .launch file, using a .xml list

once these params are set, the node should launch properly. The node publishes on the /odom_vesc topic a nav_msgs/Odometry message.

If you echo it, it will be empty, until you publish on BOTH the /commands/motor/speed and /commands/servo/position topics.

rostopic echo /odom_vesc --noarr

It will also publish a transform from base_link to the odom frame.

Steering Mapping

steering_angle_to_servo_gain = 1
steering_angle_to_servo_offset = 0.22
