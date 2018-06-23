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

  Five minutes is frequent enough for the players to consider the data up to date and by still caching the data, we are only     hitting the database once every five minutes for an important area of the game.


