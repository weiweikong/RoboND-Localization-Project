# Where Am I?
Udacity Robond Localization project

## Project Overview
Welcome to the localization project! In this project, you will learn to utilize ROS packages to accurately localize a mobile robot inside a provided map in the Gazebo and RViz simulation environments.

Over the course of this lesson, you will learn about several aspects of robotics with a focus on ROS, including -

- Building a mobile robot for simulated tasks.

- Creating a ROS package that launches a custom robot model in a Gazebo world and utilizes packages like AMCL and the Navigation Stack.

- Exploring, adding, and tuning specific parameters corresponding to each package to achieve the best possible localization results.

Note: Similar to project 1, you are required to document the work along the way. Use the project rubric as reference when you work through the project!

## Gazebo: Hello, world!
In Term 1, you worked on several projects where most of the project structure was already implemented or provided to you. In this section, you will start from scratch and develop your own package in ROS throughout this project!

[image_1]: ./images/empty_gazebo_world.png
![alt text][image_1] 

## Robot Model: Basic Setup
As a roboticist or a robotics software engineer, building your own robots is a very valuable skill and offers a lot of experience in solving problems in the domain. Often, there are limitations around building your own robot for a specific task, especially related to available resources or costs. 

That's where simulation environments are quite beneficial. Not only do you have freedom over what kind of robot you can build, but you also get to experiment and test different scenarios with relative ease and at a faster pace. For example, you can have a simulated drone to take in camera data for obstacle avoidance in a simulated city, without worrying about it crashing into a building!

You previously learned about the Unified Robot Description Format (URDF) in Term-1. In this section, you will build a very basic mobile robot model by creating its own URDF file!

### Unified Robot Description Format (URDF)
Unified Robot Description Format or urdf, is an XML format used in ROS for representing a robot model. We can use a urdf file to define a robot model, its kinodynamic properties, visual elements and even model sensors for the robot. URDF can only describe a robot with rigid links connected by joints in a chain or tree-like structure. It cannot describe a robot with flexible links or parallel linkage.

### Robot URDF
Let's first create a new folder in your package directory and an empty xacro file for the robot's URDF description.

### Launch Model

[image_2]: ./images/robot_base.png
![alt text][image_2]

Create a new launch file that will help load the URDF file.

### Robot Actuation

[image_3]: ./images/blind_robot.png
![alt text][image_3]

Great work till now! You will next add wheels to your robot. For this robot, you will require only two wheels. Each wheel is represented as a link and is connected to the base link (the chassis) with a joint. 

### Robot Sensors

[image_4]: ./images/robot_sensors.png
![alt text][image_4]

During both Term 1 and 2, you have come across and learned about different sensors that help provide robot feedback about its environment. For this robot, you will add two sensors - a camera and a laser rangefinder.

### Laser Rangefinder
Now, let's add the laser rangefinder. ROS offers support for a lot of different types of sensors. One of them, that you will be using for this robot and the project, is the Hokuyo rangefinder! 

### Gazebo Plugins

[image_5]: ./images/our_final_robot_model.png
![alt text][image_5]

You have successfully added sensors to your robot, allowing it to visualize the world around it! But how exactly does the camera sensor takes those images during simulation? How exactly does your robot move in a simulated environment? 

The URDF in itself can't help with that. However, Gazebo allows us to create or use plugins that help utilize all available gazebo functionality in order to implement specific use-cases for specific models. 


## RViz
While Gazebo is a physics simulator, RViz can visualize any type of sensor data being published over a ROS topic like camera images, point clouds, Lidar data, etc. This data can be a live stream coming directly from the sensor or pre-recorded data stored as a bag file. RViz is your one-stop tool to visualize all the three core aspects of a robot: Perception, Decision Making, and Actuation.

In this section, you will integrate your model into RViz and visualize data from the camera and laser sensors!

## Adding a map

[image_6]: ./images/localization_map.png
![alt text][image_6] 

So far, you have created a robot model from scratch, added sensors to it to visualize its surroundings, and developed a package for this robot to launch it in a simulated environment. That's a great amount of work you have accomplished!

But, what surroundings is your robot currently sensing? And how would you go about localizing the robot if you don't know where you wish to localize it? 


In this section, you will launch your robot in a new environment using a map created by Clearpath Robotics.

## AMCL
You learned about Monte Carlo Localization (MCL) in great detail in a previous lesson. Adaptive Monte Carlo Localization (AMCL) dynamically adjusts the number of particles over a period of time, as the robot navigates around in a map. This adaptive process offers a significant computational advantage over MCL. 

The ROS amcl package implements this variant and you will integrate this package with your robot to localize it inside the provided map.

You will first load the provided map using a new node for the map_server package. Previously, the robot_state_publisher helped built out the entire tf tree of your robot based on the URDF file. But it didn't extend that tree by linking in the 'map' frame. The amcl package does that automatically by linking the 'map' and 'odom' frames.

You will then add a node that will launch the amcl package. The package has its own set of parameters that define its behavior in RViz and how everything relates to your robot and the provided map so that the robot can effectively localize itself. Some of those parameters have been provided to you as a starting point. The amcl package relies entirely on the robot’s odometry and the laser scan data. 

You first remap the scan topic to the udacity_bot/laser/scan topic on which your hokuyo sensor publishes the sensor data. You defined this topic when adding the gazebo plugin for the sensor. Then, you add parameters and define values for the different reference frames such as the odom or the map frames. The amcl package will now be able to take in the laser and odom data, and localize the robot!

However, the robot is still stationary. How will it navigate to any position to collect more information from its surroundings?

That’s where the ROS Navigation Stack comes in! 

### Navigation Stack

[image_7]: ./images/navigation_stack.png
![alt text][image_7]

You will be working with the move_base package using which you can define a goal position for your robot in the map, and the robot will navigate to that goal position.

The move_base package is a very powerful tool. It utilizes a costmap - where each part of the map is divided into which area is occupied, like walls or obstacles, and which area is unoccupied. As the robot moves around, a local costmap, in relation to the global costmap, keeps getting updated allowing the package to define a continuous path for the robot to move along. 

What makes this package more remarkable is that it has some built-in corrective behaviors or maneuvers. Based on specific conditions, like detecting a particular obstacle or if the robot is stuck, it will navigate the robot around the obstacle or rotate the robot till it finds a clear path ahead. 

### Launch Setup
You have your robot, your map, your localization and navigation nodes. Let’s launch it all and test it!

As you did in a previous section, setup your RViz by adding the necessary displays and selecting the required topics to visualize the robot and also the map.

In Rviz,
- Select “odom” for fixed frame
- Click the “Add” button and

   add “RobotModel”
   
   add “Map” and select first topic/map
   
     The second and third topics in the list will show the global costmap, and the local costmap. Both can be helpful to tune your parameters.
     
   add “PoseArray” and select topic /particlecloud
   
     This will display a set of arrows around the robot. 

Note: As you discovered in the EKF lab, you can save the above RViz setup in a configuration file and launch RViz with the same configuration every time. This will make the process more efficient for you!

Each arrow is essentially a particle defining the pose of the robot that your localization package created. Your goal is to add/tune the parameters that will help localize your robot better and thereby improve the pose array.

[image_8]: ./images/launch_setup1.png
![alt text][image_8]

In RViz, in the toolbar, 
- Select “2D Nav Goal” click anywhere else on the map and drag from there to define the goal position along with the orientation of the robot at the goal position.

[image_9]: ./images/launch_setup2.png
![alt text][image_9]

Your robot should start moving towards your defined goal position!

Whoa! Those warnings don't look good. 

The amcl.launch file is throwing a lot of warnings, your map in RViz might be flickering, your robot might not be moving too well or at all etc. 

All of these are expected. We have provided you with a template for launching the necessary nodes and packages. However, it is up to you to explore some of the important parameters and tune them to effectively navigate and accurately localize the robot in this map. 

In the next section, we will cover some of the parameters that have been provided to you and some that you can consider to get started!

## Parameter Tuning
Exploring, adding, and tuning parameters for the amcl and move_base packages are some of the main goals of this project. In this section, we will cover some parameters that will help you get started, but you will get a chance to experiment with the rest and observe what effect they bring about. 

### Expected Results
Before we jump into the list, let’s take a step back and focus on what you are aiming to achieve. The move_base package will help navigate your robot to the goal position by creating or calculating a path from the initial position to the goal, and the amcl package will localize the robot. But how do we know how certain the algorithm is of the robot’s pose? 
That’s what the PoseArray, in RViz, helps us with. The PoseArray depicts a certain number of particles, represented as arrows, around the robot. 

[image_10]: ./images/expected_results1.png
![alt text][image_10]

The position and the direction the arrows point in, represent an uncertainty in the robot’s pose. This is a very convenient, albeit a slightly subjective, method to understand how well your algorithm and your tuned parameters are performing. Based on the parameters and what values you select for them, as the robot moves forward in the map, the number of arrows should ideally reduce in number. The algorithm rules out some poses. This is especially the case for when the robot is closer to walls - it is more certain of its pose because of the laser data, as opposed to when it is roaming in an open area for too long.

[image_11]: ./images/expected_results2.png
![alt text][image_11]

You should expect similar results around the goal position. Depending upon how the robot has to orient itself, it might have slightly poorer results, and that's acceptable. But overall, the PoseArray should be narrow, centered around the robot, and pointing in the right direction.

[image_12]: ./images/expected_results3.png
![alt text][image_12]

Your aim for this project is to make sure that your localization results are as certain of the robot’s pose as the above images depict. If the spread of the arrows, or particles, is too large then the robot is highly uncertain of its position, and the same goes for which direction the majority of the arrows are pointing in.

## AMCL Parameters
The amcl package has a lot of parameters to select from. Different sets of parameters contribute to different aspects of the algorithm. Broadly speaking, they can be categorized into three categories - overall filter, laser, and odometry. Let’s cover some of the parameters that we recommend you start with or details to focus on.

### Overall Filter
min_particles and max_particles - As amcl dynamically adjusts its particles for every iteration, it expects a range of the number of particles as an input. Often, this range is tuned based on your system specifications. A larger range, with a high maximum might be too computationally extensive for a low-end system.

- transform_tolerance - This is perhaps one of the most important parameters to work with and to tune properly. Like the ones before, this parameter is also dependent on your system specifications. This helps decide the longevity of the transform(s) being published for localization purposes. A good value should only be to account for any lags in the system.

- initial_pose - For the project, you should set the position to [0, 0]. Feel free to play around with the mean yaw value.
Laser

- There are two different types model to consider under this - the likelihood_field and the beam. The likelihood_field model is usually more computationally efficient and reliable for an environment such as the one you are working with. So you can focus on parameters for that particular model.

### Odometry
- odom_model_type - Since you are working with a differential drive mobile robot, it’s best to use the diff-corrected type. There are additional parameters that are specific to this type (odom_alphas), and its recommended to focus on them as well. 
All the remaining parameters and any associated details can be found on the ROS wiki. 

### move_base Parameters
While defining the move_base node in a previous section, we introduced four configuration (yaml) files. Each file corresponded to a specific functionality that aids the package. We will cover some details for each file below.

### Local or Global costmap parameters
This file consists of parameters that specify the behavior associated with your local (or global) costmap. The local costmap relies on odom as a global frame since it updates as the robot moves forward. Since the costmap updates itself at specific intervals, and aims to cover a specific region around the robot it requires its own updating and publishing frequencies, as well as dimensions for the costmap. You will notice a similar set of parameters for the global costmap.

Some of the values provided to you are tunable, and you don’t necessarily require additional parameters. You can refer to the list here as well.

### Common costmap parameters
For the list of parameters common to both types of costmaps you define which sensor is the source for observations. In our case, that’s the laser sensor. 
Aside from that, there are a few parameters that you are required to tune. You can find some great documentation on those parameters and their purpose here.

### Base local planner
We previously mentioned that the move_base package creates and calculates a path or a trajectory to the goal position, and navigates the robot along that path. The set of parameters in this configuration file customize this particular behavior. 

Here you can find a list of all the parameters that you can explore. The category of parameters corresponding to the robot configuration is especially important. Some additional parameters that you can consider -

- yaw and xy goal tolerances - These tolerances help define how close the robot’s pose can be to the goal position. In an upcoming section, we will provide you with a C++ node that will set the goal position for the project. Tuning these two parameters will play an important part at that point.

- sim_time, meter_scoring, and pdist_scale could be useful as well.

That’s definitely a lot of parameters to cover! But don’t worry, once you start observing the effects of each parameter (or set of them) on your robot and results, you will have a more intuitive understanding of each of them for this task. It’s recommended that you try to keep track of what each iteration of adding and tuning results in. It will be extremely useful for you when you write your report for the project submission.

## Testing
Going through different parameters, understanding their implications, and testing it all out in RViz will be a challenging task. That’s why we recommend you test your implementation on a shorter path rather than the entire map. 

The pose your robot starts with places it somewhere in the middle of a corridor. It is recommended that you carry out some initial tests across the length of that corridor before the robot takes a turn. This will help you figure out several aspects to improve your implementation. You can figure out if the robot gets stuck or not initially based on where it thinks are the walls with respect to its position, how quickly the robot moves and how quickly it sticks to the trajectory, and more importantly how good the PoseArray looks as the robot moves forward. Does it shrink or get worse? 

You can easily carry out these tests using the 2D Nav Goal button in RViz’ toolbar, as we covered in another section.

[image_13]: ./images/testing.png
![alt text][image_13]

The arrow, as indicated above, will represent the goal position, and the direction will represent the goal orientation for the robot.

## Launching
The above method is great for testing, but for your project submission your robot needs to navigate to a specific goal position while localizing itself along the way. In the project repo we have provided you with a C++ node that will navigate the robot to the goal position for you. You will need to create a new folder for that.

Copy the navigation_goal.cpp file to this folder. 

In order to use or launch this node, you will first need to compile it. Fortunately, ROS can handle that for you. You will have to modify your CMakeLists.txt file for that. 

Replace this file with the file present in the repo. Then,

When launching the project, after you have launched udacity_world.launch and amcl.launch, in a new terminal run the following -

The above will run the node, and you will notice your robot moving to the goal position. 

You can also display the goal position in RViz using the “Pose” display. Try it out!

Earlier we mentioned that you could create your own RViz configuration file instead of adding different elements everytime you launched the project. In the repo we have provided you with one such file that you can add to your package.

Note: At this point, we recommend that you go through the project rubric as well. It would help you if you got familiar with some of the requirements, such as the report and any images, while tuning your parameters.,

## Recap
You have done a great job so far with this project and hopefully learned a lot to be able to create your own projects for other robotics tasks! Let’s quickly recap what you covered in this project lesson -

- You learned to build your own simulated mobile robot, added sensors to it, and integrated it with Gazebo and RViz by developing a ROS package for that robot.

- Next, you integrated ROS packages like the Adaptive Monte Carlo Localization (AMCL) package and the Navigation Stack, that would allow your robot to navigate and localize itself in a particular environment or map.

- You then learned about different parameters corresponding to these packages that could help you improve your results.

## Project Instructions
You have made significant headway with your project already. There are a few more steps that you have to work on for your submission -

- You need to explore and add more parameters to improve upon your localization results. Include a screenshot of the supplied robot at the goal position.

- Once you achieve good results on the mobile robot we covered in the classroom, you will create your own mobile robot with significant changes to the model, and aim to achieve similar results for that robot! There's a lot you can experiment with here - you could try different wheels or a different laser sensor, or different drive controllers etc.

You can find more details regarding your submission in the next section. Good luck!

## Project Submission
### Steps to complete the project
1. Follow the steps outlined in this Project Lesson, to create your own ROS package, develop a mobile robot model for Gazebo, and integrate the AMCL and Navigation ROS packages for localizing the robot in the provided map.

2. Using the Parameter Tuning section as the basis, add and tune parameters for your ROS packages to improve your localization results in the map provided. Feel free to explore new parameters using the resources provided earlier in the same section. Please include the image of RViz with the robot at goal position and the PoseArray displayed. Each run may take a long time so don't forget to screenshot your robot at the goal position!

3. After achieving the desired results for the robot model introduced in the lesson, implement your own robot model with significant changes to the robot's base and possibly sensor locations.

4. Check your previous parameter settings with your new robot model, and improve upon your localization results as required for this new robot.

5. Document your work. You could use this Writeup Template. 

### Evaluation
Once you have completed your project, use the Project Rubric to review the project. If you have covered all of the points in the rubric, then you are ready to submit! If you see room for improvement in any category in which you do not meet specifications, keep working and engage in discussions with your fellow students and your mentor!

Your project will be evaluated by a Udacity reviewer according to the same Project Rubric. Your project must "meet specifications" in each category in order for your submission to pass.

### Submission
### What to include in your submission
You may submit your project as a zip file or with a link to a GitHub repo. The submission must include these items:

1. Your entire ROS package that you developed based on this Project's lesson. Your project reviewer should be able to take the ROS package as is and run it on their system.

2. The Project write-up in the PDF format that addresses all the rubric points and is based on the template linked above.

3. Images of your results for both the robot models, the one provided in the Classroom and the one you created. Each image should consist of the robot at the goal position and with the PoseArray being displayed in RViz. Both images should have your full name watermarked on them. You can include these images as part of your write-up, but for clarity purposes, it is recommended that you include them separately as well.