---
layout: post
title:  "Building a Game Engine GUI with React and Electron"
author: Ahnaf Siddiqui
tags: diamond, jdiamond, editor, gui, react, electron
---
One of the (many) drawbacks of developing a game using our own (somewhat shoddy) game engine is the lack of easy prototyping capabilities. Creating a new character requires writing code to implement the character, load its assets (textures, animations, collider shape, etc.), and spawn it in the game, all in C++. Then compile the C++, fix compile errors, compile it again, debug and fix runtime errors, and compile again. If you just want to quickly try out an idea to see how it'll look in-game, this process can be very prohibitive.

<img class="block-image" src="/assets/editor-circle.gif">

<!--more-->

### Solution

To solve this problem, I built a graphical user interface for creating game assets. The interface, inspired by Unity, consists of three windows: a list of entities (ie, gameobjects), a list of components for the selected entity, and a game window that renders the active entities in our own [Diamond engine](https://github.com/polymergames/Diamond). In the components window, you can add a render component, animator, collider, or particle effect to the entity, and as you play with the settings on these components, the entity changes in realtime in the game window. Once you're satisfied with how an entity looks, you can save its components to config files and load those files in your game. You can also load existing files in the GUI editor, change those components visually, and re-save them. This allows for rapid prototyping and development of new game characters, objects, and visual effects.

### A React-ive Interface

The GUI was developed using the Electron framework. Electron windows are just HTML pages wrapped in native desktop windows, and can therefore be styled with CSS and run client-side JavaScript. I chose to use the React library to build an interactive user interface. Some of the React components created for the Diamond Editor include the [ObjectPanel](https://github.com/polymergames/DiamondEditor/blob/master/app/panel.js) and ArrayPanel components, which automatically display the properties of any given JavaScript object as editable input fields depending on the data type of the property. A number is displayed as a number field, a string is displayed as a text field, a boolean is displayed as a checkbox, and an object is displayed recursively as an ObjectPanel. When the user interacts with any of these fields, a callback is immediately passed the current state of the object updated with that field's new input value. This information is passed on to the Diamond engine to change the corresponding component of the selected entity, resulting in an immediate update in the game window.

<figure>
	<img class="block-image" src="/assets/objectpanel.png"/>
	<figcaption>An ObjectPanel rendered by React (right) and its internal object prop as shown by React devtools (left).</figcaption>
</figure>

### Hell is Multithreaded

Keeping the GUI and the Diamond engine in sync presented a number of challenges. The editor windows are created in child renderer processes of the main Electron process. Each renderer process has its own lifecycle, but the GUI's displayed values must be kept in sync with what's going on in the engine. Initially, I used the Diamond engine's internal entity states as the "single source of truth", and sent them to the editor windows through interprocess communication (IPC) every single frame of the game loop. That's 60 IPC messages a second. In response to each of those messages, the editor windows updated the states of their React components, causing almost all of those components to be re-rendered every single time an update was received. Unsurprisingly, React could not keep up with the rate of incoming messages, and the interface was completely unusable.

It turns out that React's renderer isn't designed to run at 60 fps. In order to make the interface responsive, I reversed the direction of communication and re-architected the application to use the input fields within the GUI as the authoritative states of the game entities. Because the React components use callbacks, an IPC message is only sent when a field is changed by the user, and only for that particular component. Nothing else is re-rendered on the UI. The update message is received by the main electron process and used to update the corresponding entity in Diamond. After this change, the UI's responsiveness jumped dramatically.

The main Electron process calls functions in [jdiamond]({{ site.baseurl }}{% post_url 2017-01-02-jdiamond-and-particles %}), a JavaScript wrapper for the C++ Diamond engine, to initialize Diamond and launch the game window. This was initially done asynchronously, meaning that the main Electron process blocked on Diamond's game loop and relied on an update callback, called from within Diamond, to handle the Electron app's logic. This setup caused significant lag on the Windows version of the editor, which was resolved by creating a new thread to run the Diamond game loop.

One important consideration is that the IPC message handler in the Electron main process is on a separate thread from the Diamond game loop. This means that it's a bad idea to call a Diamond function from within the message handler to update entity state, when you have no idea what the (non thread-safe) engine is currently doing. Instead, all received update data is added to a queue, and that queue is processed from within a callback function that is run by the Diamond game loop.

### Final Thoughts

Using web technologies to build a GUI for a native game engine is a little unconventional and poses a number of unique problems. However, it has plenty of advantages, perhaps the most important of which for a small indie studio is fast development time. I had a lot of fun learning and using [React](https://facebook.github.io/react/), and would recommend it to any JavaScript developer doing frontend work. [Electron](https://electron.atom.io/) is also worth checking out if you're specifically interested in desktop development, although I am not yet convinced that web technologies in desktop apps are an ideal solution outside of shoddy indie game engine editors (I would be happy to hear your opinion).

The Diamond Editor is open source, and you can take a peek [here](https://github.com/polymergames/DiamondEditor).
