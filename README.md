# 2D_curve-to-curve_symmetry
This Python script determines the symmetric image of any given real-valued parametric curve with respect to any other given real-valued parametric pointwise-differentiable curve in the 2D Cartesian plane. The latter curve is treated as the axis of reflection. The mirror curve equations are defined by ```xMirror_t``` and ```yMirror_t``` and his name by ```mirrorName```; for the to-be-mirrored curve we have ```xToBeMirrored_q```, ```yToBeMirrored_q``` and ```toBeMirroredName``` instead. It is recommended to always reference the last published version of the script; Python 3.10+ as well as sympy, numpy, and matplotlib packages are required.

I originally conceived this idea in early 2015. The code is relatively slow because it employs sympy to perform some necessary mathematical computations. Therefore, I recommend utilizing multiprocessing for improved performance. You can define the number of concurrently active processes via the ```nProcesses``` value. For optimum performance don't exceed the number of physical cores of your CPU (x2 if SMT (AMD) or Hyper-Threading (Intel) is enabled).

The code operates by performing the following main steps:

 - Evaluating the derivative of the mirror curve at each point*.
 - Determining the intersections between each perpendicular line* to the tangents of the mirror curve and the to-be-mirrored curve**.
 - Flipping the vectors that connect the intersection points with the point on the mirror curve from where the perpendicular line was cast.

*Considering that, in general, it is impossible to compute the derivative for an infinite number of points within a finite time, the code provides the capability to define intervals of the parameter that generates the curve and adjust the point density within those intervals. You can define these values as tuples inside ```tRangeValuesList```. Beware to define ```tRangePlot``` as well, as the parameter interval used to just plot the mirror curve. The ability to selectively define the ```tRange``` parameter intervals and adjust their point density provides an efficient approach to performing the computations required for achieving a precise result. This flexibility avoids unnecessarily lengthy computation times by focusing only on the calculations necessary to attain the desired level of accuracy in different areas of the curve.

**The to-be-mirrored curve might be defined by any 2D real-valued parametric expression in Cartesian coordinates, but the existence of an exact solution set for a system consisting of an affine equation and a nonlinear non-affine equation relies on the nature of the equations and their properties. In certain cases, numerical methods or approximation techniques are necessary to find solutions. However, a system of affine equations always possesses a set of analytic solutions; that's why in the code the to-be-mirrored curve is approximated by a set of line segments. Additionally, by increasing the point density in the range of the to-be-mirrored curve parameter, we can approach the original curve arbitrarily closely, except maybe for unusual curves like the Weierstrass function and some piecewise defined curves. You can define the range and the point density of the to-be-mirrored curve parameter as tuples inside ```qRangeValueList```.

Please note that the steps mentioned above describe the basic procedure of the code, but the actual implementation involves many additional considerations and optimizations.

Finally, the code generates a plot showcasing the two parametric curves along with the symmetric curve (beware of defining appropriate x and y boundaries of the plot via ```plt.xlim(xMin, xMax)``` and ```plt.ylim(yMin, yMax)```). This plot is saved in the folder where the script was executed from. Additionally, a .csv file is generated containing the coordinates of the points of the mirrored curve, allowing you to easily export the data if needed.


~~KNOWN BUG~~ (fixed with update 1.1.2):

~~As I had to make the assumption of using real-valued symbols, there may be certain cases in which sympy might not provide the accurate value in the ```getCoeffPerp``` function. For instance, if ```yPrime_t == sp.sympify("sign(t)")```, then yPrime at tNum=0 should return ```sp.sympify("nan")```. However, due to the assumption that the expression is always a real value, it is forced to return a real number, resulting in a value of 0 instead. This may lead to inaccurate mirroring, by reflecting points that shouldn't be.~~

UPDATE 1.2.0:

The functionality of the ```generateRange``` function has been enhanced to automatically adapt the point density proportionally to the curvature of the curve at each point. To activate this feature, when calling the ```generateRange``` function, set the ```variableDensities``` argument to ```True```, and provide the parametric equations of the curve along with the variables ```tPy``` (or ```qPy```) and ```t``` (or ```q```). Additionally, to utilize this new functionality effectively, you need to specify different values for ```numMax``` and ```numMin``` when defining the ```<parameter>RangeValueList``` tuples. Every ```numMax``` and ```numMin``` couple will dictate what the maximum and minimum densities on that interval will be. Ensure that ```start``` <= ```stop```, ```extensionStart``` <= ```extensionStop```, and so on, to avoid errors. If you only want to use variable density for certain intervals, set ```numMax``` equal to ```numMin``` (or omit ```numMin``` altogether) for the intervals where you want a fixed density. By default, variable density is turned off, and ```generateRange``` is fully backward compatible, so if you desire a fixed density for all intervals, you can leave ```<parameter>RangeValueList``` and the calls to the ```generateRange``` function as in versions prior to the 1.2.0.

Automatic density adjustment can be beneficial in certain cases where higher precision is needed while keeping ```<variable>Range``` relatively small so as not to massively increase computation time. However, it may not always achieve the desired outcome and it sometimes may increase computation time depending on how the parameters are set and on the curve's curvature. Moreover, curvature is only evaluated for pointwise twice differentiable curves, so user discretion is advised.


UPDATE 1.2.4:

~~As of right now, some piecewise defined curves might lead to inaccurate mirroring.~~ (fixed with update 1.2.4)

To ensure proper symmetries for piecewise defined curves, define them with domain intervals arranged in ascending order and populate ```<parameter>RangeValuesList``` with tuples whose ```start``` (```extensionStart```) and ```stop``` (```extensionStop```) values coincide with the domain intervals boundaries of the curve. Moreover, if the to-be-mirrored curve is a piecewise-defined curve, initialize ```qIntervals``` as a tuple of sympy's Intervals and pass it as the last argument to the call of the ```segment``` function.

UPDATE 1.3.0:

Sometimes, it proves necessary to examine the mirror image for a specific set of points within ```tRange```; moreover, in certain cases, relying solely on numerical approximations for the values of these points might not suffice. To address this, the function ```addValues``` enables you to specify a list of points to be added to ```<parameter>Range```.

Please be aware that evaluating intersections for exact sympy symbolical values could sometimes be highly time-consuming. To prevent waiting indefinitely for the intersections of a single point to be computed, the ```mirror``` function has the option to include a ```maxTime``` value as the last argument of its call. The ```maxTime``` value determines the maximum duration, in seconds, for which you are willing to wait for the intersections computation to conclude when ```tVal``` is an instance of ```sp.Basic```.

This time check, although sometimes necessary, is not perfect. Since there is no direct way in Python (for non-UNIX systems) to stop a function's execution based on how long it takes to run, it would be necessary to create a subprocess every time there is the need to check the intersections between the tangent line to the mirror curve at ```tVal``` and each segment composing the curve to be mirrored. However, creating a process comes with significant overhead, so the code assumes that -- for the given value of ```tVal``` -- if the intersection calculation exceeds (or doesn't exceed) the time limit for the first segment of the curve to be mirrored, it will also exceed (or won't exceed) it for all subsequent segments. This assumption, although valid in most cases, may not hold for some rare specific scenarios. Therefore, the advice is to always initialize ```maxTime``` to ```None``` and only assign it a different value if a situation arises where setting a time limit is necessary.

KNOWN ISSUES:

Isolated points of piecewise defined curves are not mirrored.

Straight sections of the to-be-mirrored curve that are perpendicular to the mirror curve at some given points, are not mirrored if said points ```tVal```s are not present inside ```tRange```.
