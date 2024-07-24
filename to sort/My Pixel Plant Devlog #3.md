# My Pixel Plant Devlog #3

Hey guys, so as promised in my previous post, in this post I will be explaining the architecture for [My Pixel Plant](https://printygames.itch.io/pixel-plant).

## Overview

The code is written purely in [TypeScript](https://www.typescriptlang.org/)/[HTML](https://www.w3schools.com/html/)/[CSS](https://www.w3schools.com/css/), and it took me around 30 hours to write it. I'm using [Webpack](https://webpack.js.org/) to bundle my code. When writing my code, I tried to keep portability in mind to allow for easy porting to other platforms, if needed.

[Source code](https://github.com/Printy-Studios/pixel-plant/)

## The architecture

So the basic architecture of my game is like so: I have a base Game class, and that Game class has several "manager" component classes that manage different parts of the game.

The current manager classes are as follows:

 * UIManager
 * SaveManager

I am planning to refactor the Game class to also have a PlantManager that manages the plant behavior, as the code for that currently resides in Game, which viloates the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)

I know that the manager pattern has received some critique in the past, but I found that it worked very well in my game as a means of organizing and decoupling code.

In addition to manager classes, I also have several low-level modules that handle stuff like rendering and cache. These classes don't 'manage' anything, but give access to low-level functionality such as rendering and storage. I want to eventually implement an interface for each of the low-level classes, so that they can be swapped for others, for example for another renderer. This would allow for my game to be more easily portable to other platforms.

There are the following low-level classes:

 * Renderer
 * Cache
 * Storage

Additionally, I have a Plant class that holds the data and behavior of the plant.

## Low-level classes

The low level classes are the classes that handle the low level behavior such as rendering and storage. These classes are cruicial for allowing for easy porting of the game.

### Renderer

The Renderer class is the one responsible for rendering. It holds methods such as drawPlant, drawSprite, drawRect, drawProgressBar. It uses the HTML canvas for rendering. It should be possible to easily swap it for a different renderer class that uses something other than Canvas (OpenGL/WebGL) for example. I plan to accoplish this by creating a Renderer interface/abstract class that the subclasses must implement, so the game can work with any type of renderer.

[Source code for the Renderer class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/Renderer.ts)

### Storage

The Storage class is the class responsible for persistent data storage, which is the save data in the case of this game. This class uses the browser's localStorage, but just like Renderer, should ideally be swappable with other types of storage such as file storage

[Source code for the Storage class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/MyStorage.ts)

### Cache

The Cache class is responsible for storing non-persistent, per-session data. This is so that the game runs faster and doesn't have to re-download an image each time it is needed. Under the hood it is basically just an object that can be accessed through Cache's getter and setter

[Source code for the Cache class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/MyCache.ts)

## Manager classes

The managers manage different parts of the game and its behaviors

### UIManager

The UIManager class manages the UI of the game, - initializes it, adds event listeners to buttons, and changes it depending on game's data. In order to decouple UIManager from the rest of the code, I used a global event system. When a button is clicked, for example, instead of directly calling the appropriate function, the button click emits an event that can be read by the Game class, and then the Game class reacts appropriately to the event. This has drastically lowered the amount of dependencies that the UIManager requires.

[Source code for the UIManager class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/UIManager.ts)

### SaveManager

The SaveManager class is responsible for handling the saving of data. It uses the Storage class to save the data into localStorage

[Source code for the SaveManager class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/SaveManager.ts)

## Utility classes

Utility classes are the 'unit' classes that are completely independent from the rest of the game, and could potentially be reused in other games.

### GameObject

The GameObject class is the basic unit of the game. Currently it only holds a single property - position. In hindsight this class may be redundant at the current stage of the project.

### Sprite

The Sprite class holds information about a sprite, its dimensions, texture and position. Renderer has a drawSprite() method that allows you to draw a sprite

[Source code for the Sprite class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/Sprite.ts)

### Plant

This is the central, and probably the most interesting class of the game. The Plant class holds information about the plant, its water levels, stages, sprites, etc.

[Source code for the Plant class](https://github.com/Printy-Studios/pixel-plant/blob/main/src/Plant.ts)

## Data

Each plant type is stored in a `.json` file, which get fetched upon game load. It stores information such as growth rate, water levels, plant name and ID. When creating a new `Plant` instance, I pass the plant `.json` file to `Plant`'s `fromJSON()` method, and an instance gets created from that plant file. Sprites for the plants are stored in folders named after the plant IDs, and each sprite has a name such as `1.png` or `4.png`, the number indicating the growth stage.

# Conclusion

In this post I talked about the architecture behind my game [My Pixel Plant](https://printygames.itch.io/pixel-plant). I hope it was informative and that you learned something new from it. If you're interested in learning more, feel free to check out the [source code](https://github.com/Printy-Studios/pixel-plant/)

Tags:
  article, blog, devlog, gamedev, mypixelplant