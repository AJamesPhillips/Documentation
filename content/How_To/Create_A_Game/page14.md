# Summary
Although I would have liked to mess around with audio for the game, I don't really have any experience with making music, so instead I just found some that I thought fit the style of the game.

# Loading Sounds
Sounds are really easy to get started with, and I mostly referenced [Playing Sounds and Music](/how_to/playing_sounds_and_music).

The thing about sounds is that they have to be attached to a single scene, so I loaded sounds based off of where they'd need to be used:
- **app.ts**
    1. In goToStart, I loaded in the 2 sounds for the scene (music and selection sfx)
    2. I created a _loadSounds function for the game scene that just loaded the game and ending music. Then made sure to load this along with the other assets in _setUpGame.
- **ui.ts**  
For the GUI related sounds, I created a _loadSounds function and called that in the constructor.
- **characterController.ts**  
All of the character related SFX was loaded in the Player constructor.

# Playing Sounds
Playing music is really simple, but when it comes to SFX that are short, played during scene switching and paused states, it can get kind of tricky.

I spent a bit of time debugging sound issues and here are some of the tips that I have:
1. Don't use .ogg sound files. Safari does not support .ogg!
2. Sounds played when transitioning between scenes will need time to play. Since sounds are attached to a scene, if you instantly switch scenes, there's a chance that the sound won't play at all. I was able to work around this since I included a fade transition post-process. It gave just enough time for the sfx to play before switching scenes.

# Sound Usage
Aside from just playing background music and simple sfx, I had two pretty involved sections of sound.
1. Warning SFX for the sparkler  
    - I gave the sparkler a looping warning sound that started when the energy was at the 2nd to last bar. Since this was tied to the animation, I had to make sure that if the animation didn't actually complete (the player was able to reset the sparkler before it reached empty), that the sound would still stop. I did this by making sure that I stopped the sound every time *startSparklerTimer* was called.
    - Additionally, If the game were to pause while the sound was playing, we'd need to pause this as well, which is why the intervals for the sparkler animations check for *gamePaused*.
2. The character SFX
The character had a couple different sounds, but the most difficult one to manage was the looping run sfx. I had to create an observable to know when the character was running which would determine when to play/stop the sound.
```javascript
this.onRun.add((play) => {
    if (play && !this._walkingSfx.isPlaying) {
        this._walkingSfx.play();
    } else if (!play && this._walkingSfx.isPlaying) {
        this._walkingSfx.stop();
        this._walkingSfx.isPlaying = false; // make sure that walkingsfx.stop is called only once
    }
})
```
There is an extra measure of security in making sure that .stop() would not be called more than once because Safari gave a lot of audio issues when trying to do things with sounds.  
In [_animatePlayer]() we notify the onRun observable by passing in a boolean of whether to play or not. 

# Further Reading
[Extra Features](/how_to/page15)