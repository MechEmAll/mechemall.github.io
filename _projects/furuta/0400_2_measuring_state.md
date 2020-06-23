---
    project: furuta
    type: subsect
    section: Building Simulation Scene
    title: Measuring State
---

Before applying any feedback from Matlab it is first necessary to compute thet state $$ \begin{bmatrix} \theta(t) & \alpha(t) & \dot \theta(t) & \dot \alpha(t) \end{bmatrix} $$.

Coppelia provides functions to compute the joint coordinate and velocity. That is $$ q_{i} $$ and $$ \dot q_{i} $$ of the joint. However For the joint position, some numerical error have been showed up when implementing the controller. Another approach followed was to compute them manually using the transformation matrixes.

For the body **Rotating Arm** the transformation matrix M_{0}^{1 has been computed}. It is a 4x4 matrix where at the last column there is the position of the origin of the reference of the arm with respect to ground. By using simple trigonometric relations it has been possible to compute the respective angle $$\theta$$. For the **Pendulum** we computed M_{1}^{2} to get the angle $$\alpha$$.

Since the $$atan2(y,x)$$ function has been used, the angle will vary within $$[-\pi, \pi]$$. In particular, this function shows a discontinuity when the angle jump from $$-\pi$$ to $$\pi$$ when passing from the configuration with $$ x < 0 $$ and $$y-> 0^{-}$$ to the one with $$ y->0^{+} $$ and viceversa. To extend the image to the whole $$\R$$ and avoid wrapping of the angle, a counter to count the number of revolutions has been used to give more information about the system configuration. In particular the piece of code accomplishing is: 

<pre>
    <code>
actual_alpha = math.atan2(arm2pendulum[4], arm2pendulum[8]);
if(actual_alpha*previous_alpha<0) then
    if (math.abs(previous_alpha)>1) then
        if actual_alpha>0 then
            c_alpha = c_alpha - 1;
        else
            c_alpha = c_alpha + 1;
        end
    end
end
alpha_correct = actual_alpha + c_alpha*2*math.pi;
    </code>
<pre>

Finally a simple derivative has been computed to 

Customization script code:
<pre>
    <code>
function sysCall_init()
    -- do some initialization here
end

function sysCall_nonSimulation()
    -- is executed when simulation is not running
end

function sysCall_beforeSimulation()
    -- is executed before a simulation starts
end

function sysCall_afterSimulation()
    -- is executed before a simulation ends
end

function sysCall_cleanup()
    -- do some clean-up here
end


-- See the user manual or the available code snippets for additional callback functions and details

function init_measure_state()
    Pendulum = sim.getObjectHandle("Pendulum");
    RotatingArm = sim.getObjectHandle("RotatingArm");
    Frame = sim.getObjectHandle("Frame");
    arm2pendulum = sim.getObjectMatrix(Pendulum, RotatingArm);
    frame2arm = sim.getObjectMatrix(RotatingArm,Frame);
    previous_time = sim.getSimulationTime();
    previous_alpha = math.atan2(arm2pendulum[4], arm2pendulum[8]);
    previous_theta = -math.atan2(frame2arm[4], frame2arm[8]);
    previous_theta_correct = previous_theta;
    previous_alpha_correct = previous_alpha;
    c_alpha = 0;
    c_theta = 0;
end

function measure_state ()
    
    arm2pendulum = sim.getObjectMatrix(Pendulum, RotatingArm);
    frame2arm = sim.getObjectMatrix(RotatingArm,Frame);
    actual_time = sim.getSimulationTime();
    actual_alpha = math.atan2(arm2pendulum[4], arm2pendulum[8]);
    actual_theta = -math.atan2(frame2arm[4], frame2arm[8]);
    if(actual_alpha*previous_alpha<0) then
        if (math.abs(previous_alpha)>1) then
            if actual_alpha>0 then
                c_alpha = c_alpha - 1;
            else
                c_alpha = c_alpha + 1;
            end
        end
    end
    if(actual_theta*previous_theta<0) then
        if (math.abs(previous_theta)>1) then
            thetadot_abs = (math.abs((math.pi-math.abs(actual_theta)) + (math.pi -math.abs(previous_theta))))/(actual_time-previous_time);
            if actual_theta>0 then
                c_theta = c_theta - 1;
            else
                c_theta = c_theta + 1;
            end
        end
    end
    previous_alpha = actual_alpha;
    previous_theta = actual_theta;
    alpha_correct = actual_alpha + c_alpha*2*math.pi;
    theta_correct = actual_theta + c_theta*2*math.pi;
    alphadot = (alpha_correct - previous_alpha_correct)/(actual_time-previous_time);
    thetadot = (theta_correct - previous_theta_correct)/(actual_time-previous_time);
    previous_theta_correct = theta_correct;
    previous_alpha_correct = alpha_correct;
    previous_time = actual_time;
    return({theta_correct, alpha_correct, thetadot, alphadot});
end
</code>
</pre>

The function init_measure_state and measure_state will be called from Matlab through the B0Interface.