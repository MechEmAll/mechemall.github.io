---
    project: furuta
    type: subsect
    section: Mathematical Modeling
    title: Matlab Linear Model
---

After getting the matrixes A and B in Maple, the linear model can be easily exported in Matlab. The Maple code to export a data structure to be ready to be copied and pasted in Matlab is:

<code>

</code>

So we get the following variable ready to be used. To load the parameters the Matlab function FURPEN_SSR_eqns is used:

<code>
% Set the electromechanical system model
<br>eta__g = 0.85e0;
<br>eta__m = 0.87e0;
<br>k__g = 70;
<br>k__m = 0.76e-2;
<br>k__t = 0.76e-2;
<br>r__m = 0.26e1;
<br>V__m = 0;
<br>% Set the rotary arm parameters
<br>J__p = 0.23e-2;
<br>m__p = 0.125e0;
<br>L__r = 0.215e0;
<br>m__r = 0;
<br>L__p = 0.335e0;
<br>J__r = 0.23e-2;
<br>B__p = 0.0e0;
<br>tau__1 = 0;
<br>g = 0.981e1;
<br>tau__2 = 0;
<br>B__r = 0.0e0;
<br>% Load pendulum parameters
<br>FURPEN_SSR_eqns; %This runs furuta pendulum model and sets up its state space representation
</code>

The script <code>FURPEN_SSR_eqns</code> implements the symbolic equation obtained in Maple. Since we are going to design a follower for the angle $$\theta$$ we will be getting just this state so $$ C = \begin{pmatrix} 1 & 0 & 0 & 0 \end{pmatrix} $$. This function will need the symbolic expression of the matrixes A and B.

<code>
% State Space Representation
<br>A = [0 0 1 0; 0 0 0 1; 0 L__p ^ 2 * L__r * g * m__p ^ 2 / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r) -(L__p ^ 2 * eta__g * eta__m * k__g ^ 2 * k__m * k__t * m__p + 4 * J__p * eta__g * eta__m * k__g ^ 2 * k__m * k__t + B__r * L__p ^ 2 * m__p * r__m + 4 * B__r * J__p * r__m) / r__m / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r) -2 * B__p * L__p * L__r * m__p / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r); 0 -2 / r__m / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r) * (-L__p * L__r ^ 2 * g * m__p ^ 2 * r__m - J__r * L__p * g * m__p * r__m) -2 / r__m / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r) * (L__p * L__r * eta__g * eta__m * k__g ^ 2 * k__m * k__t * m__p + B__r * L__p * L__r * m__p * r__m) -2 / r__m / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r) * (2 * B__p * L__r ^ 2 * m__p * r__m + 2 * B__p * J__r * r__m);];
<br>B = [0; 0; -(-L__p ^ 2 * eta__g * eta__m * k__g * k__t * m__p - 4 * J__p * eta__g * eta__m * k__g * k__t) / r__m / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r); 2 / r__m / (4 * J__p * L__r ^ 2 * m__p + J__r * L__p ^ 2 * m__p + 4 * J__p * J__r) * L__p * L__r * eta__g * eta__m * k__g * k__t * m__p;];
<br>C = [1 0 0 0;];
<br>D = [0];
<br>% Load into state-space system
<br>sys_FURPEN_ol = ss(A,B,C,D); % Open loop system model
</code>

