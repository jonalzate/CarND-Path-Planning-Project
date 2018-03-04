# CarND-Path-Planning-Project-P1
Udacity Self-Driving Car Nanodegree - Path Planning Project
8

# Overview

In this project, we need to implement a path planning algorithms to drive a car on a highway on a simulator provided by Udacity([the simulator could be downloaded here](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2)). The simulator sends car telemetry information (car's position and velocity) and sensor fusion information about the rest of the cars in the highway (Ex. car id, velocity, position). It expects a set of points spaced in time at 0.02 seconds representing the car's trajectory. The communication between the simulator and the path planner is done using [WebSocket](https://en.wikipedia.org/wiki/WebSocket). The path planner uses the [uWebSockets](https://github.com/uNetworking/uWebSockets) WebSocket implementation to handle this communication. Udacity provides a seed project to start from on this project ([here](https://github.com/udacity/CarND-Path-Planning-Project)).

![Simulator first screen](images/simulator.png)


# [Rubic](https://review.udacity.com/#!/rubrics/1020/view) points

## Compilation

### The code compiles correctly.

No changes to cmake configuration. A new file was added [src/spline.h](./scr/spline.h). It is the [Cubic Spline interpolation implementation](http://kluge.in-chemnitz.de/opensource/spline/): a single .h file used as recommended by the module videos.

## Valid trajectories

### The car is able to drive at least 4.32 miles without incident.
I ran the simulator for 15 miles without incidents:

![15 miles](images/15_miles.png)

### The car drives according to the speed limit.
The vehicle was programmed to always stay below the speed limit of 50 mph,
there were no violations going over the speed limit for the traveled distances

### Max Acceleration and Jerk are not Exceeded.
Max acceleration is kept under 10 m/s^2 and 
Max jerk was kept under 10 m/s^3

### Car does not have collisions.
There were no collisions during the two loops the vehicle traveled.

### The car stays in its lane, except for the time between changing lanes.
The car always stayed within lane boundaries and never spent more than 3 seconds outside 
the lanes

### The car is able to change lanes 
The car is set to use the center lane as reference and always prefer to travel on
center lane, unless when there are slower cars ahead in car's lane and it is safe (based on sensor fusion) to change lanes. Always checked there is a space of at least 30 meters before changing lanes

## Reflection

The path planning for this project was done in three stages:

- Prediction
- Behavior 
- Trajectory
The first two deal heavily with sensor fusion and the third stage
makes use of the spline
Each of the above are explained below.

### Prediction
This stage uses sensor fusion to identify other vehicles around our car and make sense
of their position with respect to lanes. We can identify three different cases we need 
look out for at this stage:

- Is there a car in front of us blocking the traffic.
- Is there a car to the right of us making a lane change not safe.
- Is there a car to the left of us making a lane change not safe.

We can find the above using the current lane position from sensor fusion data and 
calculating future position at end of last planned trajectory traveled. For safety 
we assume it is dangerous to change lanes if a car is within 30 meters in front or behind 
our car in either lane.

### Behavior
This stage also makes use of the sensor fusion data, which we use to
determine what should our car do in the current situation based on prediction data.
Two main things are identified at this stage:

  - There is a car in front of us, is it safe to change lanes?
  - Should we speed up or slow down?

Using prediction data to determine our current situation, at this stage we either
increase speed, slow down or change lanes if it is safe. To keep track of our speed 
we set a flag (speed_dif) that it is then used again when planning the trajectory so we 
can adjust the speed at that stage for a better response from the car, instead of directly
adjusting speed at this stage which can give little time to react later.


### Trajectory
For planning the trajectory, I took the advise from the instructors and used the spline header to 
simplify things a little bit. This stage is heavily based on the walkthrough video for this 
project.

We first create a two lists (ptsx, ptsy) that contains widely spaced (x, y) waypoints, evenly spaced at 30m. references create to the current car's x, y and yaw and then check if we 
have any waypoints left from the last path planned that the car traveled. 
If this is the case we use our car's current x, y values as starting points.
If the previous planned path still has many untraveled points, we use those as our 
starting points. 
We then generate next waypoints evenly spaced out at 30m and add these points to the lists.

The points lists are used to initialize a spline and then we calculate future waypoints, convert
back to global coordinates, adjust the speed according to the flag from behavior stage and pass the next values to the planner for the car to execute. 

