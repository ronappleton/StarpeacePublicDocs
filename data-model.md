## The data-model

### Overview

So this document is a simple document regarding the use and implementation of the data model and some of the characteristics
we need to bare in mind for hitting the correct setup to use and the why's and wherefore's.

### Reading

Now reading is important to us as we want to make data available to the end user as quickly as we can get it to them, but on
the same note, we will also be employing a redis cache for passing that data back after the first user has requested it.
From that statement we can see that the data we would like to cache will be "general use" data, or data that is useful to more than one person, this makes sense, so in the context of the game, we can cache:

- Town data
- Tycoon data
- Planet data
- Area data

We take take that down further into those separate areas with stuff like Tycoon commercial data, data relating to the services that tycoon owns. This is great as it can be used for multiple reason, in-game by players checking out their competition, or from the website by someone simply looking through the data of a good, well ranked player.

There is however data that will not be read as often but will also not change that often, for example the data seen in the details tab of a building, a player may check the same building a few times if upgrading, changing suppliers, that kind of thing, and another player could look at that data in order to learn from it. So read not very important there as we can put a cache date of 30 days on it, if the player updates something that that affects it, we will invalidate the cache and recache anyway.

Lets look at a scenario:

User hits client, enters login details, chooses area (galaxy), chooses world, client checks user can access the world, client presents option to enter world as Tycoon (Investor) or as Visitor (Spectator) if the Tycoon is already setup on the world the client takes them back to last exit position, or if visitor, takes them into the world at some random point (capitol if exists a random town hall if not).
The user moves around a bit, looking to see what has changed on the map, then checks imports and exports to see things are running correctly, opens there company tab to see what last years taxes were, has a quick look at loans to see if they can pay any off or need to borrow anything, they know one of there HQ's needs upgrading so they open the details tab on the building, and hits upgrade.=

Thats a pretty common scenario and gives us a great way to start looking at data requirements from a users perspective, and then how we would handle that from an api perspective (specifically data we are not really interested in how at the moment, only where the data comes from and when).

So lets break it down into points.

- 1 User hits client
- 2 Enters login details
- 3 Chooses area (galaxy)
- 4 Chooses world

1. Not really worth assessing here, the person hitting the client could be Kermit the frog for all we know.
2. This is done every time a user comes back to the game, they login, there credentials are checked, as is the validity of there access token, if needed the token is refreshed, then the client is able to access the game API on the users behalf (as long as token checks out and has not been revoked).
3. Chooses Galaxy, now this is where our first consideration occurs in this scenario.
The galaxy data is nothing more than a list of planets in each area, it looks in effect as though, area wont change often, and the worlds in those areas wont change often, but the data relating to the worlds in the area will change often.

This gives us the following considerations.

    3.1. The area names and ids will barely if ever change.
    3.2. The worlds and names and ids will barely if ever change (only on add and remove world (eclipse))
    3.3. The world data such as population count, investor count and online count will change often. 

This is real interesting as we dont want to make db calls that are not needed, So in reality and all likely hood what we will do is this:

      3.3.1. We will cache the area data for 90 days, and invalidate and update on a world change within the area.
      3.3.2. We will cache the world data for 5 minutes and then update the cache.

Five minutes is frequent enough for the players to consider the data up to date and by still caching the data, we are only     hitting the database once every five minutes for an important area of the game, which is cool as there are far more important areas that need the db access.

4. Now when the player chooses the world, they have already done so based on the data already presented or because they know what they want, Again no need to find any data for them at this point.

- 5 Client checks user can access the world
- 6 Client presents option to enter world as Tycoon (Investor) or as Visitor (Spectator)
- 7 If Tycoon is already setup on the world the client takes them back to last exit position

    5.1 This is security/authorization based and as such will always be done in real time with no cache.
    6.1 There may be conditions etc that can prevent a player entering a world, like subscription, prestige and that sort of thing, because of this we do need to check against data held in the database, however, with the exception of the subscription date, that data will not change from world start, and if it does we can use invalidate and recache, we can set cache date as 90 days here very safely.
    7.1 Its easy to look at this and think that there is no need to cache this data, but hey, we have a cache and can use it, also its validity is not of real importance as we can just pop them into wherever if need be. And following the principle of reading from the db as little as possible, its no issue to constantly update this in the cache as the client window on the world changes. In fact this actually tells us we do not need to store this in the database at all, its pointless, we will just use the cache and make it valid for 7 days and theres no cach entry, we will drop them into the map in the same way we would a visitor, this will also reduce writes on the database, which is an overhead we will be happy to lose for simply recording a players last location. In honesty we could even pass off this responsibility to the client itself to store in local storage which would save even the need for hitting the cache for the location, the client can pass the locatio the same as it would if moving around the world.
    
- 8 Entering the world

    This is an important area, we pass the location we want to see, this is the central point of the view, then we hit the cache for the view window which must also be based on the zoom level we observe the world from, the rotation direction of the view must also be passed as if viewing in an envelope then the response will be different based on a 90 degree rotation, as such we only need to cache two rotations, because it is down to the client to know where to draw from the top, the bottom etcetera to display the map tiles correctly.
    Whats in the data to be passed, well, we need the buildings unique id within the game world, the buildings type id so the correct image can be displayed, the buildings condition for changing aspects of the tile, we need the location of one of the corners and we need the size accross x and y plains, this is for each building, thats enough information to display correctly and to fetch further information relating to the buildings correctly.
    This is a difficult ideal to plan as we want to minimise hits to the database as hitting the database for map information would be far too much weight, baring in mind we want to run as many world instances from the database as possible in order to minimise running costs.
    So this will probably require some clever programming from the client and server side.
    The best way I can summise this could work is by storing the map in strips or predefined blocks the idea being that in the closest zoom would would only need to load say nine predefined blocks (a block being defined as a single screen size area on the viewing game resolution.
    
    You see as a side note, the largest map we serve at the moment is 2000x2000 this gives us 4 Million map tiles or potentially 4 Million buildings, the difference is that we will only store information for map tiles that have a building on it because theres no need to store it otherwise as it will be rendered as land anyway.
    This takes into account that all buildings have a starting x and y, we do not need to store any data regarding the tiles the building covers as that it assesses when trying to build a building, if the new building would occupy a space already taken by the building at x,y that is x long and y wide then deny the building.
    So we need some theory to whittle down to a realistic amount of buildings on a built up map.
    So, say 20% of the map is water (thats 800,000 blocks we wont be storing)
    Building sizes range from 1x1 to 7x7 but the average i would say lies around 3x3
    For each building built, we should reserve say a length and width of road, so for a 3x3 building the road would be 1x6 for example. So lets just say that the av building size will be 4x4 instead and that includes road estimations.
    So within the 3.2 Million spaces (1789*1789 by my shocking maths) we could fit 1789/4 * 1789/4 buildings or 200,032 buildings, however each of the buildings will only consume 1 not 4 blocks, so we can divide that again by 4 to get a realistic building count for a fully populated map which is 50,000 buildings, so in theory we should store only 50,000 buildings on a fully populated map, this is good we have an ideal to work with now.
    out of pure interest and being real bad at maths, i am going to see the difference on a 1000x1000 map.
    - 1000*1000 gives us 1 Million tiles.
    - 20% water = 200,000
    - sqrt(800000) = 894X894 blocks
    - 894/4 = 223
    - 223*223 = 49950
    - 49950 / 4 = 12500 buildings max
    - note: i clearly did a little rounding, i dont think im wrong to do that.
    - so the estimation is 1 quarter of the map tile size for the tile map 4 times larger, ooh a correlation
    - I am sure that could help at some point but hohum.
    
    
  Ooh, so interesting discovery, if i am looking at the usual game view window envelope on a 1440 by 900 resolution
  the tile ratio appears 2:1 width:height, another insteresting fact from 2.5D.
  The width appears on this resolution to be 30 tiles, so the height is then 15 tiles
  This gives us 450 tiles on screen in the envelope which for some reason surprises me.
  At max zoom out (5 steps) it looks to give us 70:35 which in turn gives us 2450 blocks in view
  Zoomed in is 30:15  (450 blocks)
  Zoom Out 1 is 40:20 (800 blocks)
  Zoom Out 2 is 50:25 (1250 blocks)
  Zoom Out 3 is 60:30 (1800 blocks)
  Zoom Out 4 is 70:35 (2450 blocks)
  
  I wonder if we can pickup something from the block amounts maths wise.
  
  2450 - 1800 = 650
  1800 - 1250 = 550
  1250 - 800  = 450
  800  - 450  = 350
  
  Interesting indeed, the block count increases by a factor of 100 blocks at each zoom level.
  
  There is deep and meaningful maths in there, but in my very small layman head, i am guessing that if we load map data in 100 block chunks regardless of the resolution, screen size, then we will probably be doing ok.
  
  At full zoom out, we are then calling 70% of 2450 which is 18 (100x100) blocks width and 8 blocks (100x100) to fill the screen (30% height) which by my maths should be 144% of the area we need to cover which I think is perfect for overspill also.
  
However, the consideration here is that the blocks within the viewing window can be off by 5 blocks based on the 100x100 blocks.

So if we increase by one block each way so 19x9 blocks i think we should be covering 177% of the window which should cover it nicely.

Now based on that generosity of data, we should not have to worry too much about movement, as we should have a movement envelope of (needed 1715 x 735) 3035 X 1300 meaning that local movement should be round about covered and if we load an area of map into memory in the client, it will stay there. Its just working out based on movement speed and hitting the bounds of the data we have, at what point the client should request updated data, but ill leave that to the client to worry about.

My consideration is then how quick i can build the data and pass to client and cache at that point.

Obviously, each envelope we build will be passed off to cache for other users, and the fact buildings may change within it will cause the update frequency of the cache, but then, what we should be doing is removing any real time data from the transit and only passing reference data, that way we can use it to reference changes, so when a users view point changes if a building within that view changes image to construction 2, then we can pass that in the response for that area.

  
  
  
    

