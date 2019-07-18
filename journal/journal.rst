##############
Thesis Journal
##############


*Better late than never.*


Current status
^^^^^^^^^^^^^^

Simulator
*********
Nonlinear simulator working.
A lot of parameters are estimated.
Comments in source code clarify which ones are measured, estimated and guessed.
Especially lift/drag/angle of attack functions are estimates right now.

Controller
**********
I had a semi-working controller that could both hover and fly following a line between waypoints with approach speed.
(Should exists a commit with that setup).
It was all ad-hoc solutions however.

Right now I have let go of generating a good reference quaternion to do trajectory following because the thesis becomes too big with it included.

What I have:
Emil Fresks P2 quaternion controller -> translator -> actuation signals

The translator will be the focus of the rest of my thesis.

Logically one can argue that a delta thrust provides rotation around z axis.
A delta Elevon actuation provides torque around x axis, and a joint Elevon actuation generates torque around y-axis. (Yaw,roll,pitch).

I tried this in my ad-hoc controller and it works ok.
The gains for each pair of actuation signals have to be tuned not only separately but also different for different working points.

Logically: if you are flying normally forward you need a slight pitch up actuation to maintain altitude. If you then want to yaw one Elevon will appear more effective due to the increased airflow over it. => The system is coupled.

I tried to decouple them making a few assumptions:
- No winglets
- No damping effects
- No moment from lift due to camber.

This didn't work.
I had forgotten to convert angular accelleration references to torques. But even with that fixed the system is not really stable anywhere.
The translator contains division with zero when the speed approaches zero net airflow over the wing.

I then tried to remove the assumption of no moment from lift due to camber.
I also used matlabs symengine to decouple the system so that a yaw signal would no longer induce a roll.

For some reason I still have to tune the P2 controller very precicely and it is very susceptible to oscillations in yaw and roll. Pitch works great.

If I tune the torques around each axis differently I get somewhat satifiable results, but values are extreme. 100% of pitch torque, 10% of roll torque and only 1% of yaw torque.
This **should** not be necessary. The torques are normalized in my decoupler with inertias in mind.
They are, however, not normalized with damping effects in mind.
Could that be the issue?
Yaw damping at flying speeds is probably prominent, roll damping is very prominent and pitch damping is really not that big.

*Hm.*

