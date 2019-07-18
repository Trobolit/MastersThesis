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
The lift curve is actually pretty close at zero angle of attack.
The lift curve then is not too bad, right shape, right order of magnitude.
The Drag is overall higher than the indicated measurements found online, but I think it is close enough until I need to tune the finished controller.



Controller
**********
I had a semi-working controller that could both hover and fly following a line between waypoints with approach speed.
(Should exists a commit with that setup).
It was all ad-hoc solutions however.

Right now I have let go of generating a good reference quaternion to do trajectory following because the thesis becomes too big with it included.

What I have:
Emil Fresks P2 quaternion controller -> translator -> actuation signals

The translator will be the focus of the rest of my thesis.

Linearize
+++++++++

I tried to linearize the full system around a general working point.
That works fine.
But then trying to find the steady state inputs for that state didn't work.
Can't control 10+ states with only 3 inputs unless the dynamics are generally stable.
In this case they are not.

Limiting the states to only rotational states and disregarding actuator dynamics produced a trivial solution, put all actuators to zero.
Also makes sense since I haven't included pitch due to camber.
This leads me to think a decoupler might work well.

Decoupler
+++++++++

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

I disabled all saturations and observer how big the signals got.
Big.
Upon closer inspection the decoupler contain terms that are as big as 10^90+.
Divisions in the normal case takes care of this but any noise is multiplied extremely.
This is unfeasible.
I saw similar behavior when solving the feedback linearization.

Current attempt
^^^^^^^^^^^^^^^
Maybe by not forcing the actuation signals to be deltaElevon, jointElevon and deltaThrust we could instead do:

- Calculate joint thrust to keep altitude with PID as now,
- Calculate delta thrust for yaw moment
- We now have airflow over wings, so we divide the wanted pitch and roll moments by two, and solving each wing separately.
- First we solve joint deflection since it is the bigger equation.
- Secondly we solve roll given all information above.

Maybe, maybe, this removes the extreme sensitivity of the actuation signals.
I could see that a differential thrust makes the joint deflection point at which to apply the differential deflection extremely sensitive.
This is just my intuition right now.
If this works maybe one should do proper math on this?
