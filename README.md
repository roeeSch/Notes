# General Notes



## ROS

#### running ROS on 2 machines

In order to run see\read\display topics between two machines run the following commands:

on master machine  (where roscore runs) run:

1. `$export ROS_IP=ip_of_master`

on client machine (not where roscore runs) run:

1. `$export ROS_MASTER_URI=http://ip_addr_of_master:11311`
2. `$export ROS_IP=ip_of_client`

**<u>note:</u>** without the export of ROS_IP, only meta data of messages is available.