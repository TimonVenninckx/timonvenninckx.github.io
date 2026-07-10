# Procedural levels in Summoning Stones


## Introduction
For my fourth and final block in my first year at [BUAS](https://www.buas.nl/) we had to work in teams and create a complete game in 8 weeks and put it on [Itch.](https://buas.itch.io/venture)\
This was my first time working on a game in a bigger team, this was a really fun experience and taught me a lot about teamwork and the other disciplines: design & art. I worked mostly on gameplay systems for the actual duration of the block, implementing the quest system, minion interactions, inventory system, initial UI setup and other things based on designer mock-ups. After the official 8 weeks of the project passed some students kept working on the project and that's when I started working on procedural level generation with designer [Thijs Werkman](https://www.linkedin.com/in/thijswerkman/). This is what the rest of the blog will be about😁(this took ~3 weeks, not full time)
Make sure to play our game [**on Itch!!!!!!!**](https://buas.itch.io/venture)
## Trailer - made by Erva
{{< youtube id=2qvpzq5UhN8  >}}


## Procedural level goals
Before creating the procedural levels I needed some reference levels to determine what kind of procedural methods I would need to use to generate such levels. Since we already had a complete level in our game I could replicate elements of this level inside my procedural level generator.\
My goals were for all levels to be playable & completable and to be similar to the main level. Levels should be progressed in a linear way, with options to open side doors and be able to backtrack faster if needed. Levels should also spawn different quests and resources each time.
## Room Generation
#### Layout
Since the level should be laid out on the table and rooms should be generated in a linear way, to calculate proper costs of doors & resources count accumulating over time I chose to generate the room locations on a spline.
![initial rooms](/images/summoning_stones/pcgstep1.png)
#### Wall layout Voronoi noise
Then I create a grid where I fill the cells between the rooms, I later use these cells in Unreal Engine's PCG system, where I can then procedurally place meshes within these cells. (PCG is quite finicky to setup and especially to make sure meshes don't go out of the grid cells)
![grid generation](/images/summoning_stones/pcgstep2.png)
#### Doors generation
Now that we have walls we should generate doors connecting the rooms, so before I send over all the filled grid cells to the PCG I step on the grid between the rooms that should have doors to each other. Each room has a door to the next room, and the room after that (only if the rooms are close enough to each other). And for some nicer backtracking I hardcode a door between room 1 and 5. As can be seen in the next image.
![grid generation](/images/summoning_stones/pcgstep3.png)
#### Puzzle pieces
Now we had some rooms with a grid where I can place resources to gather inside of. And barricades can be spawned on the pathways between rooms, cool!\
![grid generation](/images/summoning_stones/small_room_off.png)\
However this was quite empty/boring in practice, as in the main level you had all sorts of cool small rooms which had been authored by the level designer, and can look much better than my randomly generated walls.
This can especially be seen when playing the game here's an example without small rooms:\
![grid generation](/images/summoning_stones/small_onground_off.png)\
Here's an example with small rooms turned on, these have resources or pebble piles inside of them.
(Pebbles are the minions that follow you and do tasks for you)\
![grid generation](/images/summoning_stones/small_onground_on.png)\
I called these small rooms puzzle pieces. You can easily create them by creating a child blueprint, placing some decoration inside and correctly placing the placeholder node and cobweb. As long as you make sure the cobweb is oriented towards the scene root😄. 
When placing these pieces I can rotate them 90, 180 or 270 degrees, which looks quite good. This is the parent BP to inherit from:
![grid generation](/images/summoning_stones/parent_puzzlepiece.png)\
And also an example:\
![grid generation](/images/summoning_stones/skull_room.png)

#### Piece placement
So, how do we place these pieces? Some constraints: They should NOT spawn inside of blockades, they should NOT spawn on the edge of the table and they should be oriented towards the center of the room so they can always be walked inside of. Otherwise they could be blocked by walls.
I did this by attempting to spawn a piece from the center of each room a couple times. Then I check a different grid which has the placed doors and the already placed puzzle pieces as filled positions. This grid looks like this:\
![grid generation](/images/summoning_stones/pcgstep4.png)
\
In red you can see all the pathways, and the puzzle pieces that have been spawned.
Now we take the old grid which has all the walls:\
![grid generation](/images/summoning_stones/pcgstep2.png)
\
And we remove all of the pathways and puzzle pieces, and end up with this:\
![grid generation](/images/summoning_stones/pcgstep5.png)
\
Then we spawn our resources, some lights, and doors and end up with this😁\
![grid generation](/images/summoning_stones/pcgstep6.png)

## Balancing
Balancing, determining how many resources are spawned, and if there should be a "quest completion place", or "minion buy place" is all determined based on randomness and previous rooms. Quests are generated first, determining how many resources are available inside of the room. Then we determine how many minions can be bought inside of the room (if any at all).
Then with the left over points we decide the pricing of the next rooms, then we iterate over all the rooms and create a 100% completable level, add some power scaling and randomness to the quest generation and you have some interesting procedural levels!


## PCG
I used Unreal Engine's PCG system to spawn a bunch of meshes to create walls between each room. The system was honestly quite painful to work with, as it was not intended to use on a bunch of grid-like points and especially making sure that meshes fit reasonably inside of the bounds given by the grid was hard. I did end up with a reasonable solution, however It doesn't look that good in my opinion because it's hard to use a lot of bigger assets like the main level does. But PCG is very nice to use when you are instead doing open worlds, and have some big area you need to populate with trees, grass and such. Final result:\
![grid generation](/images/summoning_stones/pcgstep7.png)
## Afterword
I would like to do some more reading on how other games handle procedural content generation, as most of the things I did these last couple of weeks were just "I can try this" moments.
According to feedback, a lot of people said it played like the main level, which is nice to hear, but of course I couldn't match the level of exploration the main level gives you, since handcrafted exploration levels are hard to beat.
If you read this far, thank you for reading!!! There are many small details I have glossed over, but if you have any questions shoot me a message down below or somewhere on LinkedIn or Email (both are on the homepage).
[**Play the game on Itch!!!!!!!**](https://buas.itch.io/venture)






