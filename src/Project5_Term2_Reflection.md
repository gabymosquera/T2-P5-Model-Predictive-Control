# Self-Driving Car Nanodegree Project 5 - MPC Controller

This repository contains all of the code needed to compile and run a MPC controller on the Udacity provided Term 2 simulator. The goal for this project is for the MPC controller to successfully guide the simulator car a minimum of one lap around the track. The car shall not leave the drivable portion of the road and/or cause danger if someone was to be in that vehicle and it wasn't just a simulator.

[//]: # (Image References)
[video1]: ./video/MPC_Video_Gaby_Mosquera.mp4

#### Source Code

The src folder contains all of the .cpp and .h files needed to run the project, they are the following:

1. main.cpp
2. MPC.cpp
3. MPC.h

The starter code provided by Udacity points out several TODOs within main.cpp and MPC.cpp. Completing these are vital for a functioning controller and successful completion of the project. Typically Model Predictive Controllers are of dynamic type, which takes into account a lot of external and internal forces for the vehicle. For this project we focussed in the kinematic model instead, which is a very simplified version of the dynamic model and still gives very good results with a lot less work. The kinematic model focuses on the steering angle and the throttle (acceleration) of the vehicle.

#### Running the Code

To run this code please follow the directions in the original link to the project repository below:
[CarND-MPC-Project](https://github.com/udacity/CarND-MPC-Project)

## Model Predictive Controller (MPC)

### The Main Goal

The main goal of our kinematic MPC is to calculate the best value to steer and throttle a vehicle and optimize such value after each time step, taking into account a path given in global coordinates, velocity, orientation, cte, and error psi.

### The Steps

#### 1) Convert given x and y points
Path planning is needed for an MPC. We are given x and y points of the planned path and we must convert these from global coordinates to car coordinates to be able to use them within the simulator. This convertion happens in lines 113 to 122 of main.cpp

#### 2) Determine coefficients for a 3rd order polynomial to fit the road
For this project we use a 3rd order polynomial, mainly because it fits most typical roads. To get the coefficients for such polynomial we need to use the provided polyfit function using the x and y points provided to us in waypoints. This coefficients will be used later on with the Solve function to calculate the optimazed values for the steering angle and the throttle of the vehicle. Refer to line 125 in main.cpp. 

#### 3) Use the kinematic vehicle model equations
To predict the state of the x and y points, velocity, psi, cte, and epsi we can use the kinematic vehicle model equations from Lesson 19.6. I implemented these in lines 136 to 143 in main.cpp. In my implementation, these equations will also take into account the latency of the model (line 136 in main.cpp).

#### 4) Use the 'Solve' function
We will feed the state vector (created using the kinematic vehicle model equations mentioned in step 4) and the coefficients from step 2 to the solver function 'Solve'. The solver will used the state values, contraints, and cost function to find a vector of control inputs that will minimize the cost function. Then the solver will use IPOPT to apply the results to the simulator vehicle and optimize again in a constant loop. Refer to line 145 in main.cpp.

#### 5) Use the optimized values
The solver is used and the results stored in 'vars' (line 145 in main.cpp). Then 'vars' is assigned to the steering and throttle (lines 150 and 151 in main.cpp). Those two values are the optimal predicted values for the steering angle and the throttle and they are fed to the simulator.


### Implementing MPC.cpp

## Timesteps and Duration
In MPC.cpp we had several TODOs as well. starting from the very important setting of N and dt. N was the number of timesteps for the model. dt would be the duration of each timestep. It was taught that the multiplication of these variables should not be more than a couple of seconds total since, in a real life situation, driving conditions can change drastically in a matter of seconds and not updating our behavior for too long would mean a potential accident.

## Parameter Tunning
The cost constraints were specified in the operator function within MPC.cpp (lines 47 to 122 in MPC.cpp). These cost constraint functions where multiplied by different parameters to give them either higher priorities or to penalize certain behaviors. These parameters where stored in a weights array called 'w' (line 54 in MPC.cpp). It quickly became evident to me that the highest the cte the more the car wobbled around the center line so the weight for epsi ended up a little higher than the cte to fix that issue. Other weights like the steering angle delta_start needed high priority to be able to control sharp turn scenarios.

Also, keeping a low value for the weight of velocity helped control the car a lot better. At first I tried values like 10 or 5, quickly changed that to 1 or below and finally found that a value of .15 really helped constrain the speed and keeping control of the car.


### Video 

Below is a video of the resulting car motion using MPC in the Term 2 Simulator:

[Video](./video/MPC_Video_Gaby_Mosquera.mp4)

The car stays within the road lines at all times and completes at least one full lap around the track.

### Challenges

1. Velocity tunning was a challenge for me. I could not figure out how to make the model work at large velocities and the best result I got was the car driving at 26 mph even using a reference velocity of 60 mph.
2. Understanding the power of tunning the cost functions with different weights was a game changer for me during this project. Increasing the weights for the cte and the epsi really improved the resulting MPC and therefore the behavior of the simulator car.
3. There were also small warnings during compilation about comparisons between signed and unsigned integers whithin the for loops in the MPC.cpp file. The solution was to simply use 'uint' instead of 'int' for the counters. Fixing this really helped the compilation speed.
4. Every time I tried using my computers screen recorder within the simulator, the simulator would immediately start to act up and would make the car go off road. Therefore I had to record a video with my phone and download such recording for the purpose of this submission.