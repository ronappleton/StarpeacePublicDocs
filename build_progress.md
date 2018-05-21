# Build Notes

This documents will form the log of what is happening as its happening, we will endevour to update this at the end of any 
spell of programming to let you know how things are going, if something has worked, hasn't worked etc.

Think of this as a journal to the development process.

# Engine 1

Engine one was about getting something happening, at the moment, its a learning process as much as a development process, we need to learn how to make this as we progress and the logical starting point was based on population generation and distribution.

Because this is a conceptual process at the moment, the code base is PHP, not because of any intention to use it for prduction, as the production engine will be coded in c#

We began by creating a couple of classes, one for engine, one for business and one for farm, the intention was to grasp how we could create a method for pulling in population and then distributing that population.

We began by creating two farms, and giving them both a requirement for 100 low class workers, 10 middle class workers and one high class worker, we also gave the classes storage for current workers.

As a basic start we set an influx rate of population each round based on a random number generator to generate a percentage based on the level of required workers. 

This was as primitive as it could be, we set a maximum percentage increase of 25% and we then generated a random number between 10% as a minimum influx and 25% as the maximum influx. This worked exactly as expected. We generated different percentages for each class on each round.

We then split the population increase equally between the two farms.

While this engine has not acheived a great deal, what it did do was give us a base grounding of how we think about generating population.

# Engine 2

Ok so for this engine we wanted to improve on the method used for sharing population by taking into account each of the businesses specific needs to incorporate the idea that different businesses can have different levels of class workers required and some businesses may not need workers of a certains class.

We did this by incorporating variables to hold data on the class needs of each business and separately storing how many businesses needed workers of a certain class.

We continued to share the poulation out evenly between thoughs that needed it but this time incorporating the count of businesses needing the class of worker, we also added code for handling the division being odd and for reducing the amount of businesses needing a class of worker, when the share had been made. That way we could still share the whole population influx between the businesses needing the workers.

The results were identicle to the last engine but with more logic in place. We also introduced data tables for displaying what was happening each round as the enabled us to see the effects of tweaking also.

# Engine 3

This time round, we wanted to handle the fact that different businesses have different pay scales and because of that the desirability between two businesses could be different, because of this we looked at deciding factors of sharing population between the businesses.

We did this by introducing the concept of average pay. We realised that if we calculated the average pay being given on each round then as prosperity grows, so will pay expectations and disparity between different businesses pay levels, because initially all businesses have the same wages levels, we set the startup average as 1 (one) and then from each round moving forward we calculate it by totalling and then generating an average of pay level.

This was then used to decide how much of a population increase each of the businesses should recieve each round, we consider this acceptable as the more pay a business pays, the more likely they are to attract workers, it also means that in the game, you would actually be able to steal other players workers by paying them more, which again is more realistic than the current methods where higher wages means you fill your required workers quicker, but does not have a baring on workers moving on into better paid positions.

Thus we have introduced the concept of prosperity into the population itself as later on we can introduce a class status variable and if a low class worker for example is above the average pay for middle class workers in that town, they will be upgraded to middle class.

We incorporated this by calculating the business pay level against the average wage level to be able to generate a percentage of share the business should receive and we were pleased that testing immediatley showed the power of average pay, as Farm2 (lc wage level 5) had its low class workers filled by round 7, where as Farm 1 (lc wage level 1) took a further 70 rounds before hitting full employment.

# Engine 4

WE are happy with progress so far but we believe that pay disparity should also have an effect on the population generated each round, as if average wages are higher in one town, then that town should pull in more population than a town with a lower average wage level.
