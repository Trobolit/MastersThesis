##############
Thesis Journal
##############


*Better late than never.*

.. contents::

Current status
^^^^^^^^^^^^^^

10/9
****
Got to speak to a guy that actually developed the PX4 support for matlab, along with some supervisor.
We got pretty much everything working again, except the actual logging when running in standalone mode.
Probably going to have one more meeting for that.

9/9
***
Still have not heard back from Mathworks.
Kind of need to get the log files working, else I am in real trouble here.

I added an altitude holder to the EPO wing, that worked wonders.
Had my first successful landing thanks to it.
Then I burnt out both motors.
Turns out the extra load on them due to my not as efficient 3D printed props pushed them over the limit.
I already had them running at double the voltage intended with a super hefty ESC (50A+ I think) killed it in the end.
Now I am transitioning to the much bigger motors intended for the final EcoSoar.
Just need to 3D print a mount to get them attached properly.
I also have much better props for these motors.


5/9
***
Back in Malawi.
I brought a small EPO foam flying fixed wing with dual rotors to do the initial experiments on, before I transition to the big EcoSoar.
A crude variety of the real controller (no airflow correction since propellers cover entire elevons) was implemented and it hovers after some initial tuning.
Now that I wanted to look at the logs and start to analyze data turns out Matlab has a bug preventing me from getting any data of the drone.
Have filed a support ticket with them.

Once I can get the data off of it I plan to do steps in all directions, then look at that data.
Should it look acceptable I will try to fly it through a transition.
This would be a proof of concept for Emils P2 controller on a tail sitter through a transition with no switching modes.

Then I will build and test the full airflow compensating controller on an EcoSoar.
Hopefully things work great.
If not it can be many things, but most likely the torque translator parameters will need heavy tuning.
If i don't run out of time I could maybe try to implement a parameter estimator that find the correct parameters for me during flight.
That might even cancel some errors on the fly due to simplifications I have made.


Previous
********
The pixhawk can be programmer directly from simulink.
It requires one to jump through some hoops, but works well.
I have rebuilt the bixler aircraft, attached a gps, pixhawk, etc to it and built my own small live-tunable PID fly by wire system.
Just to get experience on the setup on an aircraft which is ok to crash.
I have permission to fly at the local university following few weeks so hope to get some hours down.
I do need, however, to implement my own MAvLink protocol in simulink for it to work with QGroundControl.
It is still faster than learning to make my own custom firmware for the pixhawk based on ardupilot.

I also added wind to the simulator.
The quaternion reference generator works acceptable in it, but could use some work.

**Update**:
I rebuilt the Bixler with a Pixhawk onboard.
I added a GPS to it and managed to start the built in MAVlink protocol handler.
I had to recode some of the Simulink blocks to support all channels, correct data type, etc.
But now it all works acceptable.
The built in handler only provides location, heading and attitude. Nothing else.
QGroundControl does, however, manage to extract these things and show them in real time.
I had very little success manually sending data over the serial link.
I built a small serial send/receive program onto the pixhawk and connected the ground station to my linux laptop.
This verified that the Pixhawk does not like to send data for some reason, and receiving is terrible.
If I spam the sender with raw bytes from my laptop I can make them show up in the Pixhawk, but only once I overfeed the link.
So making my own MAVlink implementation from scratch would mean rebuilding the Serial HAL (hardware abstraction layer) in the firmware, probably.
I did some tinkering with the firmware configuration and found the built in MAVLink handler.
This worked, but I still haven't successfully added/read/written anything to it from simulink.

I now know that the MAVlink antenna onboard the aircraft is responsible for the servo jitter.
May need to build a shield or something.
I've read online that some people managed to turn down the gain of the antenna in the firmware. Might want to try that. 

**Furthermore**
I build a small FBW (fly-by-wire) control scheme as a simple test, implemented it and flew.
The first attempt resulted in the worst crash so far.
Turns out I accidentally reversed the reference input signal for pitch...
The controller adjusted correctly but when I wanted to pitch up the controller thought I meant pitch down...
Rebuilding the aircraft, re-uploading the new code, and the airplane flies very well.
This is then a proof of concept; I can make my own control schemes that rely on the underlying state estimators already in the firmware, in Simulink, build and upload directly from Simulink (even though it is extremely slow), and fly them.

I think this is good enough for this thesis.
Later I might improve it all, add missing features etc, but now it is time to build the dual engine aircraft, make it hover using my simple torque translator, Emil Fresks P2 controller and try a few small step inputs.
Compare that with the simulator and we should be good to write a report.

-----
All written below has been tried.
The smulator now works good enough I think.
I will start learn how to program the pixhawk.
The torque translator turned out to be pretty straight forward once:
- The wings are treated separately given the flow over each wing
- The static pitch moment is not accounted for, but inputted as separate constant. This corresponds to the actual hardware trim we do in reality to achieve zero pitch moment at zero actuation in flight. In normal linear flying theory it actually is a constant, because things cancel in the equations, but in our non-linear case it actually isnt a constant. It does, however, seem to not vary too much. Little enough that once the baseline is set the controllers can handle the disturbances.

I maybe should try to add wind distrubanes and wind gusts to the controller.
But I think, to keep things realistic, I will scale away all waypoint things for the tests in reality and just have the altitude controller, P2 controller and torque converter.
This means that I can make a small simulator for just that (given a constant or step quaternion) to compare with reality.
It also limits the complexity of the tests, which is a good thing; things are already very complex.
With this smaller simulator disturbances do not have to be modeled as wind or gusts, I can just add them directly to the rigid body equations in the simulator, maybe as steps, noise, etc.

Now I need to start:
- (Write down the equations for the current setup with tunable parameters where appropriate)
- Learning how to, and program the pixhawk
- Cad and 3d print the changes to the aircraft
- Build the two aircraft
- Do simulations that resemble the actual test I will perform
- Do the experiments, tune and make things work
- Compare to simulator, maybe adjust simulator to make it behave as reality (if changes are feasible)
- Write report on it all.

Then if I have time left over I may try to either implement my controller to take reference signals from the already existing waypoint system, or try to implement my own so that the local entrepeneur working with these can use them in the field.

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

**Update:**
The controller works, sort of.
I tuned it during simulations and had a full sucessful flight with hover take-off, normal flight, and hover-land.
I still need to tune the different axis separately, but I think that makes sense.
Each rotational axis represents quite the different system from the others.
45 degrees off in yaw is not too bad, and is allowed to take time to get back, but 45 degrees off in pitch is terrible.

So. Plan now is to rerun simulations but have the torque-translator block output what would have been the effective gains to a scope for inspections.
I also have problems with oscillations and bang-bang happening around pitch sometimes.
Will look into that as well.
If this entire solution seems promising afte rmuch scrutiny I may have found a solution to the P2-translate-actuation signal problem.
Now I just need to define it mathematically, try to work out my solution with maths, implement it in c-code, etc.

*Misc Notes*:
The matlab command ``simplify`` is a major player in the induction of really big numbers into the equations.
Because the equations contain decimal numbers from physical parameters (otherwise the equations would probably become way too big, I might try to work around this later) it can shorten the equations by finding common denominators.
This leads to multiplications generating these huge numbers.
Yes the length of the equation is shorter, but the simplifications do not make real sense.
