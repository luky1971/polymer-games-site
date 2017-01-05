---
layout: post
title:  "Particles, and a JavaScript API for Diamond"
author: Ahnaf Siddiqui
tags: diamond, jdiamond, particles
---
We're finally back with another blog post! We've been making solid progress on [Polymer]({{ site.baseurl }}{% post_url 2016-07-02-first %}) in the months since our last post (gameplay footage coming soon!), but today I'd like to talk about a couple other things I've been working on for our [open-source game engine](https://github.com/polymergames/Diamond).

<img class="block-image" src="/assets/jetcircle.gif">

<!--more-->
The scene above was created using [jdiamond](https://github.com/polymergames/jdiamond), the node.js wrapper for our own Diamond engine. To make that scene, all you have to do is `npm install jdiamond`, create a game.js file, and write some code:

``` js
const Diamond = require('jdiamond');
const fs = require('fs');

if (Diamond.init()) {
    // calculate where we want to spawn our ship
    const pos = Diamond.Vector2.scalarVec(
        Diamond.renderer.resolution, {x: 0.5, y: 0.25}
    );

    // build the ship
    const shipSprite = Diamond.renderer.loadTexture("laserShip.png");
    const laserShip = new function() {
        this.transform = new Diamond.Transform2(pos);
        this.renderer = new Diamond.RenderComponent2D(this.transform, shipSprite, 1);
    };
    laserShip.renderer.pivot = {x: 40, y: 95};

    // make a cool particle effect
    // and attach it to the ship
    const particles = new Diamond.ParticleEmitter2D(
        JSON.parse(fs.readFileSync("jetstreamparticles.json")),
        laserShip.transform
    );

    const movespeed = 1;
    const turnspeed = 0.2;
    // this function will be called every frame
    const update = function(delta) {
        // move the ship!
        laserShip.transform.position.add(
            Diamond.Vector2.rotateVector(
                {x: delta * movespeed, y: 0},
                laserShip.transform.rotation * Diamond.Math.DEG2RAD
            )
        );
        laserShip.transform.rotation += delta * turnspeed;
    }

    // start the game loop
    Diamond.launch(update);
    Diamond.cleanUp();
}
```

Of course, you also have to have the ship and particle sprites, and the .json file with the particle system settings. All of this and more can be found [here](https://github.com/polymergames/jdiamond/tree/master/demos). To run your game just do `node game.js`.

If you aren't familiar with node.js, what all this basically means is that it is super easy and fast for JavaScript developers to get started making PC games with jdiamond.

So what's the point of having yet another JavaScript game engine? Unlike most JavaScript game libraries, jdiamond does not target the web (HTML5 and Canvas). Instead, it uses a C++ rendering and physics engine to make high-performance desktop games that run on native OpenGL. And unlike some desktop engines that claim to feature "JavaScript" (coughunitycough), jdiamond is a true node.js package that can be installed with a simple `npm install jdiamond` and take advantage of the full breadth of node.js libraries and features.

You may have noticed that our game loads the settings for the particle system from a JSON file. This is what that file looks like:

``` js
{
    "particleTexture": "assets/monomer.png",
    "minParticlesPerEmission": 1,
    "maxParticlesPerEmission": 10,
    "minEmitInterval": 10,
    "maxEmitInterval": 20,
    "minParticleLifeTime": 1000,
    "maxParticleLifeTime": 1000,
    "minEmitPointX": -10,
    "minEmitPointY": -10,
    "maxEmitPointX": 10,
    "maxEmitPointY": 10,
    "minEmitAngleDeg": -180,
    "maxEmitAngleDeg": 180,
    "animateScale": 1,
    "minBirthScale": 0.07,
    "maxBirthScale": 0.07,
    "minDeathScale": 0,
    "maxDeathScale": 0,
    "minParticleSpeed": 0,
    "maxParticleSpeed": 0
}
```

Besides these settings, the particle engine also allows you to set acceleration values and color animation (not yet implemented). We also plan on adding more emitter shapes like circles and polygons.

If we change the settings to something like this:

``` js
{
    "particleTexture": "assets/monomer.png",
    "minParticlesPerEmission": 30,
    "maxParticlesPerEmission": 30,
    "minEmitInterval": 10,
    "maxEmitInterval": 10,
    "minParticleLifeTime": 1000,
    "maxParticleLifeTime": 1000,
    "minEmitPointX": -100,
    "minEmitPointY": -100,
    "maxEmitPointX": 100,
    "maxEmitPointY": 100,
    "minEmitAngleDeg": -180,
    "maxEmitAngleDeg": 180,
    "animateScale": 1,
    "minBirthScale": 0.07,
    "maxBirthScale": 0.15,
    "minDeathScale": 0,
    "maxDeathScale": 0,
    "minParticleSpeed": 1,
    "maxParticleSpeed": 1
}
```

we get a particle system that looks like this:

<img class="block-image" src="/assets/bigfountain.gif">

Admittedly, my real motivation behind developing a JavaScript API was so I can use it to create a Diamond editor GUI using the awesome [Electron](http://electron.atom.io/) framework. Having a GUI will make it easier to create even cooler particle effects for Polymer- that's what I'll be doing next, so stay tuned!

If you want to try out jdiamond, read the installation instructions [here](https://github.com/polymergames/jdiamond) to get started. It's still very early in development, and lacks major features like input and animations. We appreciate any feedback- and if you're interested in contributing to the project, let me know! Leave a comment below, or find us on [Twitter](https://twitter.com/polymer_games) or [Facebook](https://www.facebook.com/polymergames/).
