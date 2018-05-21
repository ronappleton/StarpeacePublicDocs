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

The results were identicle to the last engine but with more logic in place. We also introduced data tables for displaying what was happening each round as they enabled us to see the effects of tweaking also.

# Engine 3

This time round, we wanted to handle the fact that different businesses have different pay scales and because of that the desirability between two businesses could be different, because of this we looked at deciding factors of sharing population between the businesses.

We did this by introducing the concept of average pay. We realised that if we calculated the average pay being given on each round then as prosperity grows, so will pay expectations and disparity between different businesses pay levels, because initially all businesses have the same wages levels, we set the startup average as 1 (one) and then from each round moving forward we calculate it by totalling and then generating an average of pay level.

This was then used to decide how much of a population increase each of the businesses should recieve each round, we consider this acceptable as the more pay a business pays, the more likely they are to attract workers, it also means that in the game, you would actually be able to steal other players workers by paying them more, which again is more realistic than the current methods where higher wages means you fill your required workers quicker, but does not have a baring on workers moving on into better paid positions.

Thus we have introduced the concept of prosperity into the population itself as later on we can introduce a class status variable and if a low class worker for example is above the average pay for middle class workers in that town, they will be upgraded to middle class.

We incorporated this by calculating the business pay level against the average wage level to be able to generate a percentage of share the business should receive and we were pleased that testing immediatley showed the power of average pay, as Farm2 (lc wage level 5) had its low class workers filled by round 7, where as Farm 1 (lc wage level 1) took a further 70 rounds before hitting full employment.

# Engine 4

We are happy with progress so far but we believe that pay disparity should also have an effect on the population generated each round, as if average wages are higher in one town, then that town should pull in more population than a town with a lower average wage level, and on the same note, the lower the wages of a town the less likely hood there is of attracting population to the town.

Now as the code base develops, more and more factors will be introduced which will influence population growth, so this a again in interim measure to facilitate growth but in a reasonable way at this moment in time.

This is a super hard issue to resolve, we have established that there should be no fixed lower or upper amount of population influx, that sounds like a simple idea to field, but for us it was not, generating something from nothing really is hard. At the same time we need to consider that people may not always want to come to the town, there could be no jobs, all the jobs could be low paid, and crime could be high, we need to bare this in mind as it may also mean that we need to remove population also, and should that happen, we need to decide where that loss of population should come from.

At this point we have decided that we will do the population movement once a day, or twenty four ticks, as we wish to make ticks equivilent to an hour, then we have a simple principle to have faster and slower worlds by halving or doubling the ticks respectively.

We have established that the influx should be based on historical figures to prevent fast fluctuations based on some abusive behaviour that may corrupt the simulation.

Ok, so sounding radical we have come to the conclusion that the initial minimum growth rate should be 0% at engine start and the maximum influx at engine start should be 100%, these rates are based on if nothing is built, if theres no reason to come to a town, then people wont. We will be adding a chaos fluctuator at a later date to make this seem a little more organic, take settlers for example, sometimes people setup there lives in remote places because they can, and we should incorporate that.

To develop the growth rate we have decided that each desirability factor that causes town desire should have a fixed percentage and that negative effects should also have a percentage that is fixed, what we shall do is calculate the total percentage for desire and then subtract the negative percent to get the total percentage of needed population we will influx during the population tick.

So initially we will give businesses requiring workers equal to a maximum input percentage of 20%, we know this will drop as we add desirabilities but for now 20% will do just fine to introduce the principle.

We will need to some calculations and at this point realise that the amount of calculation we need to do is rising so we are going to introduce a class for doing out calculations for us.

Ok so changes for this engine complete, because of the extra thought put into the calculation it was success first time round.
There was a small issue in that I put an empty check instead of a not empty check, but the result was exactly as hoped.

We moved to introducing population at every 24 ticks only which extended the time it took the buisnesses to fill to around 700 ticks,next we tried setting farm 2's wages to 3 per tick for low_class workers compared to farm 1's 1 wages per tick, and the result was it took around twice as many ticks to completely fill both buildings, which we expected as the desirability would have reduced nicely, with Farm 2 filling up by tick 440 and Farm 1 fill eventually by round 1390 something, this is a great result, if nothing else i

# Engine 5
## Residential Desirability
The intention of this engine will be to introduce the concept of residencies and introduce the concept to the generation AND distribution of the population influx.

For example a business with closer residentials should attract staff easier than a business with residentials further away. We should also incorporate the fact that if the residential building is full then that desirability should be removed, and also for example a business needs 30 workers but residentials only have space for 20 of them.

# Engine 6
## Supply and demand
This engine will be a real step forward as we will introduce the concept of the trade centre, raw products, and finished goods ready for retail, it will also introduce the commercial business to retail that product, this will be a simple introduction of the concept though as we want to start factoring in the population buying products in the next engine.

