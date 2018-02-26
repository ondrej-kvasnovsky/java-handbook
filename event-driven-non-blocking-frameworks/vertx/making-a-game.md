# Making a game

We are going to make a realtime multi-user game using Vert.x.

> This example is inspired by [real time bindding with websockets and vert.x](http://vertx.io/blog/real-time-bidding-with-websockets-and-vert-x/) article.

## Game Rules

The game principles will be:

* a hero can request to join the game
* when required number of heroes is present, it starts the game
* there are two teams and heroes are placed on opposite sides when the game starts
* when the game starts, treasures are placed on the map but they are hidden underground
* when players start playing, they can go and fight others with shovels, which is not very effective
* or they can go and use the shovel to dig out a treasure, there are various artifacts that can be found in a treasure, like excalibur sword, powerful cursed magical wands or epic armor from dragon skin. 
* when they dig in the ground, they can find things like dirt, worms, coins
* when one side wins, the get all collected things and they split it amongs the winning team, losers get nothing
* heroes can use the spoils to buy special equipment to make their players look ultra cool

Ok, I think I just freaked out with these requirements, but that could be nice game to play I think :\) Lets just create a playground and let the heroes move there.  
![](/assets/start-game.png)

## Setup Project

We will need two parts. The first one is the frontend and then backend that will provide data and share data amongst players. 

_WIP_

