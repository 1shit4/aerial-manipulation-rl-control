# Robust Goal-Directed Aerial Manipulation

**Reinforcement learning and disturbance-compensating control**  
*Robotics Research Center, IIIT Hyderabad | 2025–2026*

> This repository provides an overview of the research project and links to its experimental implementations across simulation, reinforcement learning, and flight control.

## Overview

The objective of this research was **accurate and robust control of an aerial manipulator**: coordinating a quadrotor and its robotic arm to bring the **end effector to a desired pose** while maintaining stable flight under dynamic uncertainty and external disturbances. Accurate end-effector control is essential for aerial pick-and-place, where arm motion and payload changes can significantly disturb the flying base.

The quadrotor and manipulator are dynamically coupled. Arm motion changes the system's mass distribution and introduces forces and moments that affect the flight dynamics. These effects are difficult to model precisely and can compromise both vehicle stability and end-effector tracking accuracy.

A **hierarchical learning-and-control approach** was investigated, with complementary responsibilities:

- **Reinforcement learning (RL):** generate coordinated commands for the **quadrotor and manipulator** from the robot state and desired end-effector goal.
- **Disturbance-compensating inner-loop control:** track the generated references while responding to fast, uncertain dynamics arising from manipulator motion, payload variation, and modeling errors.

The research covered RL action-space design, policy training and sim-to-sim evaluation, aerial-manipulator task development, disturbance-compensating control, and controller-aware RL training. The repositories below contain the corresponding experimental code.

## Research Components

### RL control-output selection

An important design question was **what level of control output should be learned by the RL policy**. Quadrotor hovering and position-control experiments in **Genesis** explored:

- **Direct rotor RPM commands:** the policy specifies individual rotor speeds.
- **Collective thrust and body-rate (CTBR) commands:** the policy specifies total thrust and desired angular rates, while lower-level control handles rate tracking and motor allocation.

The Genesis implementations also explore reward formulations, action smoothing, simulated actuator and controller imperfections, and domain randomization. Related quadrotor RL experiments were developed collaboratively in **Isaac Sim/Isaac Lab**.

### Quadrotor RL training and sim-to-sim evaluation

**Flightmare** was used to train goal-conditioned quadrotor policies for hovering and low-speed position/trajectory tracking, including line, circle, square, and figure-eight references. Exported policies were evaluated through **ROS 2/PX4–Gazebo** interfaces to test their behavior outside the training simulator. Limited experiments also applied a quadrotor-trained policy to an aerial-manipulator model.

### Goal-conditioned aerial-manipulator control

For aerial manipulation, the control target is the **end effector**, requiring coordinated motion of the vehicle and arm rather than vehicle position tracking alone.

**Aerial Gym** implementations include aerial-manipulator task configurations with vehicle and joint states, end-effector kinematics, and goal-conditioned rewards. One implemented policy interface uses **10 action components**:

- Desired vehicle acceleration: `ax, ay, az`
- Desired body rates: `p, q, r`
- Desired yaw: `yaw`
- Desired arm-joint positions: `joint_1, joint_2, joint_3`

This interface differs from the earlier CTBR-only quadrotor experiments. Supporting code translates the policy references into lower-level flight and manipulator commands, with separate ROS 2/PX4 inference and tracking implementations for Gazebo.

### Inner-loop control and disturbance compensation

Alternative controllers were implemented and evaluated in **ROS 2/PX4–Gazebo** to compensate for disturbances associated with manipulator motion:

- **Disturbance-estimating control (ATDC-style):** estimates lumped rotational disturbances and compensates for them in the commanded body torque.
- **Adaptive sliding-mode control (ASMC):** uses attitude and angular-rate errors with adaptive feedback terms to compute corrective torque.
- **Incremental nonlinear dynamic inversion (INDI):** uses angular-acceleration feedback and a control-effectiveness model to calculate motor-command corrections.

Controller experiments include waypoint and trajectory tracking, including circular and square paths, with programmed arm-joint motions to introduce disturbances. The controller repository provides the control laws, input/output definitions, and PX4 interfaces.

### Controller-aware RL training

**Controller-in-the-loop RL training** was explored in Aerial Gym so that policy outputs would pass through the low-level controller during training, rather than only at inference. The controller's response affects the simulated vehicle motion, which in turn determines subsequent observations and rewards. This allows the learning environment to account for the closed-loop dynamics encountered when the controller is used during execution.

## Code Repositories

The repositories below contain **different experimental implementations developed toward the same research objective**. They cover distinct simulator environments, control interfaces, and integration work rather than forming a single packaged project release.

| Component | Repository | Contents |
| --- | --- | --- |
| Genesis RL experiments | [drone_rl_genesis](https://github.com/1shit4/drone_rl_genesis) | Rotor-RPM and CTBR interfaces, hovering, reward variants, and randomization |
| Isaac Lab experiments | [quadcopter_issaclab-rl](https://github.com/rocon-rrc/quadcopter_issaclab-rl) *(lab repository)* | Collaborative quadrotor RL implementation in Isaac Sim/Isaac Lab |
| Flightmare training | [flightmare](https://github.com/1shit4/flightmare) | Quadrotor PPO training, testing, and model export |
| Flightmare inference | [RLQuadcopterControlFlightmare](https://github.com/1shit4/RLQuadcopterControlFlightmare) | Exported-policy inference through ROS 2/PX4–Gazebo |
| Aerial-manipulator simulation | [arm_control_ws](https://github.com/1shit4/arm_control_ws) | Vehicle/arm simulation, joint actuation, and Gazebo integration |
| Inner-loop control | [quadcopter_inner_loop_control](https://github.com/1shit4/quadcopter_inner_loop_control) | Disturbance-estimating, ASMC, and INDI controller implementations |
| Aerial Gym training and integration | [aerial_gym_simulator](https://github.com/1shit4/aerial_gym_simulator) | End-effector task environments, learning configurations, and controller-in-the-loop code |
| Aerial Gym inference | [RLquadcopter_aerial_gym](https://github.com/1shit4/RLquadcopter_aerial_gym) | ROS 2/PX4 policy inference and tracking experiments |

Supporting [PX4-Autopilot](https://github.com/1shit4/PX4-Autopilot) and [PX4-gazebo-models](https://github.com/1shit4/PX4-gazebo-models) forks contain customized vehicle/manipulator models and configurations. The underlying simulators and flight-control frameworks belong to their respective upstream projects.

## Publication

Work from this research project contributed to **[Reinforcement Learning with Inner-loop Dynamics Estimator for Aerial Manipulation under Uncertainty (IEEE CASE 2026)](https://arxiv.org/abs/2606.16621)**.

*Research conducted collaboratively at the Robotics Research Center, IIIT Hyderabad.*
