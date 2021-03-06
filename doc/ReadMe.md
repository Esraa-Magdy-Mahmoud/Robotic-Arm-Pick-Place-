
# Project: Kinematics Pick & Place
## Kinematics Analysis 


### 1-Forward Kinematics demo
![fk](https://user-images.githubusercontent.com/17046622/35038031-0e2fb7c6-fb82-11e7-803a-8a6249a7e2f7.png)

make the Robot model semi-transparent by assigning alpha to 0.3

### 2-DH Parameter table
Links|alpha(i-1)|a(i-1)|di   |theta_i     
--- | --- | --- | --- | ---
0_1  |0         |0     |0.75 | theta_1    
1_2  |-pi/2     |0.35  |0    |theta_2-pi/2
2_3  |0         | 1.25 |0    |theta_3     
3_4  |-pi/2     |-0.054|1.50 |theta_4     
4_5  |pi/2      |0     |0    |theta_5     
5_6  |-pi/2     |0     |0    |theta_6     
6_G  |0         |0     |0.303|0             

* The Parameters are derived from URDF file and with the help of the figure below showing the DH convention while taking into account that the convention of the URDF file is diffrent from the DH convention.
* Given the DH Params, a transofrmation matrix from i to i-1 can be calculated. 
* Transformation from the end-effector to the base link will be a composition of transformation
    T0_g = T0_1 * T1_2 * T2_3 * T3_4 * T4_5 * T5_6 * T6_g

*  The URDF doesn't folow the same convention of the DH parameters,Hence the frames of the DH model will not be the same as Rviz/Gazebo to Compensate for rotation discrepancy between DH parameters and Gazebo a rotation matrix involoves rotation about z axis with 180 degs then about the y axis with -90 degs is implemented.
The following two images illustrate the DH parameters frame convention versus Rviz frame convention.

1-Rviz Convention
![rviz_conv](https://user-images.githubusercontent.com/17046622/35038197-b63cb81a-fb82-11e7-8276-fe9cd65712ca.png)
2-DH Convention 
![dh_conv](https://user-images.githubusercontent.com/17046622/35038151-8a156ade-fb82-11e7-9696-65bca57f7ec4.png)



### 3-Decouple IK Problem nto Inverse Position Kinematics and inverse Orientation Kinematics
first, To calculate the Wrist Centre pose with respect to the base_link. a Compositions extrinsic Rotations using Euler angles multiplied by the correction rotaion matrix to correct convention are applied.
The comopsition of the mentioned rotations will result in the following matrix:

Rrpy = R_z * R_y * R_x * R_corr 
then WC_position = EE_pose - d7*R

Second, Calculate the first three joint variables.


The Calculations of the last three joint angles can be done by extracting the rotation matrices from DH Transformations.
The resultant rotation is:
R0_6 = R0_1*R1_2*R2_3*R3_4*R4_5*R5_6
since we calculated the rotation from the base_link to the end effector then Rrpy = R0_6

Then for the last three joints R3_6 = inv(R0_3) * Rrpy 
**Note : for a rotation matrix the inverse of the rotation is the same as its transpose,but calculating the treanspose is not computationally expensive as the inverse.**

from R3_6 the joint angles are found as follows :

theta5= atan2(sqrt(r13*r13 + r33*r33), r23)

to take care of multipe solutions :

    if sin(theta5) < 0:
        theta4 = atan2(-r33, r13)
        theta6 = atan2(r22, -r21)
    else:
        theta4 = atan2(r33, -r13)
        theta6 = atan2(-r22, r21)


**Notes on Ik_server.py***
I did all the calculations once in IK_debug and saved all the outputs into files, so when running the code only needs to look up into the files not doing any complex matrix multiplications.

the Ik solution while loading the pre-calculations from files can be found in approximately less than 0.2 secs

![out](https://user-images.githubusercontent.com/17046622/35038190-b0e99c2a-fb82-11e7-8032-fe1e966c5eea.png)



# 4- IK_server.py
the code starts with defining the symbols for the joint variables and the yaw, pitch and roll angles.After that, it looks into the files where the pre-calculations are saved and retrives them. 
The pre-calculations involves the DH Parameters dictionary and all the matrices needed in symbolic format, including the total transformation from the base link to the end-effector, the correction rotation matrix, the yaw-pitch-roll rotation matrix and the rotation matrix from base link to the end-effector.
Then , it extracts the position and orientation of the end-effector and calculate the wrist centre and joints angles as mentioned in the previous section.

