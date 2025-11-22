# Operating Guide

## Hardware Connection
- **Note 1:** Since the OBU upper computer is configured to send data to the lower computer via “usb0”, after connecting the OBU, ensure that the network interface is recognized as **usb0**.  
  If it appears as **usb1**, a reboot may be required!  
  ![usb1](./images/usb1.png)

- **Note 2:** The current UDP reception configurations of the two OBUs (query via AT commands) are:  
  - Device ID **22019446**: `0,192.168.62.224:30301`  
  - Device ID **21029792**: `0,192.168.62.223:30301`  

  The current UDP sending configuration for both OBUs (query via AT commands):  
  `2,usb0:30299` (where **2** indicates TLV mode)

- **Note 3:** Cooperative perception equipment:  
  - **Sending side:** OBU **22019446**, Orin **22017658**  
  - **Receiving side:** OBU **21029792**, Orin **22017657**

- **Note 4:** For V2V communication, **both OBUs’ GNSS indicators must be flashing blue**, otherwise communication will not work!

---

## Experiment Workflow

### Sending Side
Enter the workspace directory `ros2_workspace` and source the installation file:
```
. install/setup.bash
```

If the code has been modified, rebuild the ROS2 package:
```
colcon build --packages-select point_cloud_infer
```

ROS2 node code location:  
`ros2_workspace/src/point_cloud_infer/point_cloud_infer`

Launch files location:  
`ros2_workspace/src/point_cloud_infer`

Sending-side launch file:  
`bounding_box_client_launch.py`

Sending-side ROS graph:  
![sender_rosgraph](./images/sender_rosgraph.png)

This graph helps understand the relationships and message subscriptions between nodes.

Start the sending-side nodes:
```
ros2 launch point_cloud_infer bounding_box_client_launch.py \
  pcd_path:=/home/thu/Downloads/2021_08_23_21_47_19/243 \
  engine_path:=/home/thu/Downloads/PointPillarNet/checkpoint_epoch_80_fp32_v2.engine \
  rate:=2
```

Open Wireshark and select **usb0**.  
You should see communication from the upper computer **192.168.62.223** to the lower computer OBU **192.168.62.199**:
![Wireshark_sender](./images/Wireshark_sender.png)

---

### Receiving Side
Similarly, enter the `ros2_workspace` directory and source installation files:
```
. install/setup.bash
```

If the code has been modified, rebuild the ROS2 package:
```
colcon build --packages-select point_cloud_infer
```

Receiving-side launch file:  
`bounding_box_server_launch.py`

Receiving-side ROS graph:  
![receiver_rosgraph](./images/receiver_rosgraph.png)

Start the receiving-side nodes:
```
ros2 launch point_cloud_infer bounding_box_server_launch.py \
  pcd_path:=/home/thu/Downloads/2021_08_23_21_47_19/255 \
  engine_path:=/home/thu/Downloads/PointPillarNet/checkpoint_epoch_80_fp32_v2.engine \
  rate:=2
```

**Note:**  
To simulate real cooperative perception, launch both sending and receiving side simultaneously to ensure spatiotemporal alignment.  
Ensure both OBUs' GNSS LEDs are flashing blue.  
If they are flashing blue but the receiving side Wireshark does not capture packets, you may need to **restart the Orin** (reason currently unknown).

Open Wireshark on the receiving side and select **usb0**.  
You should see communication from lower computer OBU **192.168.62.199** to upper computer **192.168.62.223**:
![Wireshark_receiver](./images/Wireshark_receiver.png)
