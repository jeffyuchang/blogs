--- 
layout: post
title: Back To Basics -  Review notes on Robotic Software Architecture Course
tags: 
- Back to Basics
- Artificial Intelligence
- Robotics
---

![Robotic and World](../../../../images/robotic-world.png)

As the above picture shows, the robot senses the world from its sensor, and then do some actions to the world. We can typically think of the world as a Map too make it not too abstract.

When dealing with robotics, we need to deal with uncerntainty and nosiy, for example, when you asked robot to turn left 90 degree, it could just turn about 85 degree, we called it having noisy, or maybe robot didn't move at all due to mechanic problems and the sensor is not perfect either, so we need to add the noisy to that too. Therefore, we need to take the nosiy into account when we are creating the model for it.

##### Bayesian Localisation #####
Assume the robot doesn't know its location, and then wants to localise himself in the world(e.g map), here we have some landmarks in the map that robot can sense via its sensor. It has two updates: *Motion Update* and *Observation Update*.

- [Bayes Rule](http://en.wikipedia.org/wiki/Bayes'_theorem):
   P(S|O) = P(O|S) * P(S) / P(O) == likelihood * prior / evidence. Here S means state, O means observation.
- [Kalman Filter](http://en.wikipedia.org/wiki/Kalman_filter):
   It is a special case of a Bayes’ filter with dynamics model and sensory model being linear [Gaussian](http://en.wikipedia.org/wiki/Normal_distribution). 
- [Extended Kalman Filter](http://en.wikipedia.org/wiki/Extended_Kalman_filter):
	When the sensory model is nonlinear, then we may use this filter to make it like linear.
- [Particle Filter](http://en.wikipedia.org/wiki/Particle_filter):

*Note*:[Here is a nice rundown article](http://swarmlab.unimaas.nl/wp-content/uploads/2012/07/fox2003bayesian.pdf) with regard to these filters, and here are three series on understanding the Extended Kalman Filter (EKF.): [PartI](http://services.eng.uts.edu.au/~sdhuang/1D%20Kalman%20Filter_Shoudong.pdf), [PartII](http://services.eng.uts.edu.au/~sdhuang/Kalman%20Filter_Shoudong.pdf) and [PartIII](http://services.eng.uts.edu.au/~sdhuang/Extended%20Kalman%20Filter_Shoudong.pdf).

##### Search #####
Search Problem is just model. Firstly, we map the problem to Search problem into following properties: __States, Actions, Successor Function and Goal test__. Below are some Search Algorithms(b denotes the branching factor, m denotes the tree's depth for the time complexity and space complexity):

- [Depth-First Search](http://en.wikipedia.org/wiki/Depth-first_search):  Takes time O(b^m ), Spaces that fringe take O(bm)
- [Breath-First Search](http://en.wikipedia.org/wiki/Breadth-first_search): Takes time O(b^m ), Spaces that fringe take O(b^m )
- [Uniform Cost Search](http://en.wikipedia.org/wiki/Uniform-cost_search): This one is typically used in the weighted tree. Strategy: expand a cheapest node first. Fringe is a priority queue(priority: cumulative cost)
- [A^* Search](http://en.wikipedia.org/wiki/A*_search_algorithm): The priority queue is ordered by f(n) = g(n) + h(n). Here the g(n) denotes the cost that already took, the h(n) means the hurestic cost to the goal. This algorithm is a cornerstone of search, and has been used in a lot of fields.

##### Graph Search #####
Idea: never expand a state twice

Implementation:

- Tree search + a set of expanded states ("closed set")
- Expand the search tree node-by-node, but...
- Before expanding a node, check to make sure its state is new
- if not new, skip it

In above algorithms, we are always dealing with deterministic situation, for cases involved non-determinisc (stochastic), we will use the __Markov Decision Process(aka MDP)__.

##### Markov Decision Processes #####
[MDP](http://en.wikipedia.org/wiki/Markov_decision_process) is used to find a optimal way in the non-deterministic(stochastic) search area. It uses the [Bellman Equation](http://en.wikipedia.org/wiki/Bellman_equation#Example) to do the calculation. 

![Bellman Equation](../../../../images/mdp_bellman_equation.png)

Here is the denotation that we used in the formula:
 __S__ -> state, __R__ -> Reward, __A__ -> Action, __O__ -> Obersavation, __T__ -> Transition Function.
 
- Value Iteration:
    The formulation is as following:
    ![Value Iteration Bellman Equation](../../../../images/mdp_vi.png)
- Policy Iteration:
	The formulation is as following:
    ![Policy Iteration Bellman Equation](../../../../images/mdp_pi.png)
- Prioritized-Sweeping:
> Use a Priority Queue to update nodes in a smart order, on each update, track |ΔV|, and add it to the priority of all upstream states. Reset priority to 0 when a node is pulled from the queue and updated.

MDP applies to the fully observable, [POMDP](http://en.wikipedia.org/wiki/Partially_observable_Markov_decision_process) technique applies to the Partical Observal case. In both MDP and POMDP we assume that we know the __Transition function (T)__ and __Reward Function (R)__, in the cases that we don't know them, it will lead to the next section, Reinforcement Learning.

##### Reinforcement Learning #####
Because we don't have the T and R function, we can use the following techniques:

- [Temporal Difference Learning](http://en.wikipedia.org/wiki/Temporal_difference_learning)

   ![TD](../../../../images/rl_td_algorithm.png)
- [Q-Learning](http://en.wikipedia.org/wiki/Q-learning) (aka. Model free technique)

   ![Q learning](../../../../images/rl_q_learning.png)

##### Optimisation #####

- [Gradient Descent](http://en.wikipedia.org/wiki/Gradient_descent)
- [Function Approximation](http://en.wikipedia.org/wiki/Function_approximation) 
- [Downhill Simplex Algorithm](http://optlab-server.sce.carleton.ca/POAnimations2007/NonLinear7.html)
- [Powell's Method (Conjugate Gradient)](https://wiki.ece.cmu.edu/ddl/index.php/Powell's_method)

##### Planning #####
- [State space planning](http://en.wikipedia.org/wiki/State_space_planning)
- Plan space planning

##### Summary #####
Above is a list of bulletins that were taught in this course, guess it is a bit hard for others to follow due to its simplicity, most likely it is for myself use. :)

#### Reference ####
- [UNSW CS3431 Robotic Software Architecture Course](http://www.cse.unsw.edu.au/~cs3431/wiki/)
- [Introdcution to Artificial Intelligence](https://www.ai-class.com)
- [CS373 Programming A Robotic Car](http://www.udacity.com/overview/Course/cs373/CourseRev/apr2012)
- [CS188.1x: Artificial Intelligence](https://www.edx.org/courses/BerkeleyX/CS188.1x/2012_Fall/about)
- [Artificial Intelligence, foundations of computational agents](http://artint.info/index.html)
- [CS3431 Study Note](http://comp3431.wikidot.com)
