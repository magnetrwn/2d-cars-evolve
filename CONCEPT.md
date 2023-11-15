# Implementation Guideline

**This file contains details on what the application should do, to follow through during implementation.**

## Libraries

Definitely:
+ Box2d

Some options for rendering the actual window graphics:
+ Raylib
+ SDL2

## Car Creature Implementation

**Rigid-body**

This is how each car creature will be structured:
+ List of **hinges**, connected by lines, making up the rigid-body as the **polygon** itself.
+ Each hinge can be:
    + **Empty**.
    + **Free** Wheel.
    + **Rotor** Wheel.
+ Wheels can be circular, rectangular, triangular, with **random** size.
+ The largest width, height or radius of a wheel is **max clamped** to the longest line of the rigid-body.

**Car Creature State**

Something is needed to base the measure of car performance:
+ Car creature **mass**, which is just the total wheel plus rigid-body area total.
+ **Distance** from start.
+ Current **speed**, to determine if the car is stuck (or going backwards) and cut evaluation.

And some other values are required for evaluating car survival and fair comparison:
+ **Fuel**, with amount directly correlated to the car rigid-body polygon area, and used to allow hinges to rotate.

**State Application**

Fuel will be used for car evaluation instead of time, so that when fuel is depleted, simulation will move on to the next evaluation.

Assuming car hinges always push maximum torque, we can simulate fuel consumption dependently on mass (estimated, in this case total polygon area of the rigid-body plus hinges) and on angular velocity of motorized wheels.

## Simulation Implementation

**Evaluation Algorithm**

Evaluating each car creature will make use of parameters used in Car Creature State earlier:
+ **Distance Traveled**: Use the distance from the start of the course.
+ **Fuel Efficiency**: Calculate `(initial_fuel - remaining_fuel) / initial_fuel`.
+ **Speed Performance**: Increment a penalty metric over time in case of negative acceleration, adding its value towards a negative result.
+ **Mass Efficiency**: Negate the car mass.
+ Having specified all of these metrics, calculate the **weighted sum** of them with weights appropriately chosen after testing. This will be the result.

Evaluation is performed when one of these conditions is met:
+ Once fuel is exhausted, perform evaluation and move on to the next car creature.
+ If speed is null or negative for more than 4 seconds, perform evaluation immediately.

**Genetic Algorithm**

The simulation will use a genetic algorithm to favor better-performing car creatures getting more chances to mix with similarly good performing creatures:
+ A total of **36 cars** will be simulated and evaluated **per generation**.
+ The last **12 cars** will be **dropped**.
+ An additional column will contain the **difference between the current car evaluation with the lower one, called eval delta**.
+ Cars with **very close eval delta will be dropped, by keeping the first** of the block of close eval deltas, **unless 12 cars have already been dropped this way**.
+ The remaining cars will merge into the next generation:
    + Pick hinges from Car1 corresponding to **33% of Car1**, and subtract them from Car1.
    + Pick hinges from Car2 corresponding to **33% of Car2**, and subtract them from Car2.
    + Pick hinges from the **remaining pool of hinges across Car1 and Car2**, with chance to pick 67%, and always apply a slight mutation to all its descriptors.
+ Apply a **very very small mutation to all hinges of all cars** in the next generation.
+ **Generate extra car creatures from scratch** to compensate the missing amount, back **to 36 cars**.

## Terrain & Obstacle Implementation

**Terrain Generation**

Randomly generated terrain follows a progressive approach:
+ Be ready to generate a square block of terrain, with constant length.
+ **Vertical Shift**: Have a vertical shift value **change the vertical position of the top-right vertex** of the square, clamped within min and max to a reasonable value.
+ When visually reaching the end of the square, generate a new block the same way, making sure the top-left vertex has the same vertical position as the current terrain block.
+ Delete old terrain blocks.

**Side Note**: It's also possible to use a **clamped rate of vertical shift** to prevent building uncrossable hills, or make the blocks rectangular instead of square, and generate their length randomly to make for more randomic terrain. This can also aid generating equivalent random terrain for each generation (although it's probably best to use the same terrain for each car creature, before moving to the next generation?)

**Obstacle Generation**

There should be two standard obstacles generated:
+ Pyramid of triangles
+ Stairs of squares

It's best to use a small terrain area with flat angle to place the obstacles, but that is not certain yet.