# Emil Forsén Portfolio
 
An overview over the projects I've made or been a part of in reverse chronological order. Click on a header to see parts of the code with reasoning and from there you can go to the full repo if you're interested in _everything_.
### Projects

>**Note**
>#### A note about Project management
> 
>I can throw around a bunch of project management terms like scrum, agile, trello, kanban boards etc. as much as the next guy - but it's basically just name dropping tools. What I did was making sure people kept their focus, encouraged them to help each other out and keeping the weekly goal concrete. This is something I've been responsible for in all of the projects I've been part of and in the interest of keeping things DRY I'll just refer back to this note when needed. 

# Individual Projects


![Beef system title.png](https://github.com/emilxf-0/Portfolio/blob/main/Images/Beef%20system%20title.png)
## [Beef System (Master Thesis)](https://github.com/emilxf-0/Portfolio/tree/main/Beef%20System))

<details open>
 
 >**Developed:** 2023 October - Ongoing
>
>**Library:** SDL2
>
>**Language:** C++
>
>**Genre:** Master Thesis
</details>

### Overview
The Beef System is a spin on the Nemesis system, made famous by Shadows of Mordor. As the name implies it's not so much a game as it is a system for emergent narrative. With that said, I wanted a setting that felt thematic so it takes place in the most infuriating of places: traffic. 

![Accio title.png](https://github.com/emilxf-0/Portfolio/blob/main/Images/Accio%20title.png)

## [Project Accio](https://github.com/emilxf-0/Portfolio/tree/main/Project%20Accio)

<details open>
 
 >**Developed:** 2023 Jan - 2023 Feb
>
>**Engine:** Unity
>
>**Language:** C#
>
>**Network:** Firebase
>
>**Genre:** Multiplayer, tug of war, pattern matching
</details>
### Overview

A 1 v 1 Android game trying to simulate the classical "two wizards shooting a colorful death ray towards each other but they are both equally powerful so they end up in sort of an inverse tug of war"-trope. I wanted to do something cool with pattern matching and trying to utilize firebase for a real time duel game.


# Group Projects

## Get Up
![[Get up title.png]](https://github.com/emilxf-0/Portfolio/blob/main/Images/Get%20up%20title.png)

<details open>
 
 >**Developed:** 2023 September - 2023 October
>
>**Engine:** Unity
>
>**Language:** C#
>
>**Genre:** VR, Single player, Walking simulator
</details>

### Overview
Get up is a VR walking sim set at the bottom of a pit. The goal is simply to walk to the top by balancing on some preeety sketchy-looking boards. 

### My contribution

**Vertical movement and foot tracking**
We wanted to explore spatial movement and the challenges of mitigating motion sickness due to the disconnect between perceived and physical movement. To do that we had to find a way for the player to move around in a limited space but still get the benefits of a relatively large game world. By letting the player move in a spiral motion we could keep the need for space to a minimum. VR doesn't play very nicely with collision since there is no way to stop the player from just moving indefinitely (barring physically restricting them, thankfully there are laws in place to stop of us from doing that) which usually means the world gets offset and mess up the placement of the game world. 

**Project management**
See note [at the top](#projects)
### Team: 

**Programmers** | **Artists** 
-------|-------
Me | [Alvin Alvrud](https://www.artstation.com/alvrudart)
[Adam Hjelm](https://github.com/Adam-Hjelm) | [Tom Hammar](https://www.artstation.com/tomhammar)
[Henrik Nyström](https://github.com/sweviceroy) | [Emil Carlsson](https://www.artstation.com/sandratollefsen)



## Flesh and Stone

![[Flesh and stone title.png]](https://github.com/emilxf-0/Portfolio/blob/main/Images/Flesh%20and%20stone%20title.png)

<details open>
 
 >**Developed:** 2023 March - 2023 May
>
>**Engine:** Unreal 5
>
>**Language:** Blueprints
>
>**Genre:** Single player, 3rd person, Hack n slash
</details>

### Overview

Flesh and Stone is a super stylized hack n slash game in the vein of DMC and God of War. We wanted to do something that had the same speedy feel like Doom but that heavily incorporates melee combat. 

### My contribution

**Combat system** 

I was responsible for making combat feel crunchy, impactful and intuitive. That meant setting up key bindings and blue prints that handled both singular attacks and combos. I worked extensively with timing and to gracefully add interactivity to the animations made by the animator. 

**BluePrints** | **Animation** | **In Game**
--- | --- | ---
![AttackInputBP.gif](https://github.com/emilxf-0/Portfolio/blob/main/Flesh%20and%20Stone/AttackInputBP.gif) | ![LightAttackAnim.gif](https://github.com/emilxf-0/Portfolio/blob/main/Flesh%20and%20Stone/LightAttackAnim.gif) | ![LightAttackClip.gif](https://github.com/emilxf-0/Portfolio/blob/main/Flesh%20and%20Stone/LightAttackClip.gif)


**Project management**

See [note above](#projects)

### Team: 
**Programmers** | **Artists** 
-------|-------
Me | [Tom Hammar](https://www.artstation.com/tomhammar)
[Max Petersson](https://github.com/Max-Petersson) | [Jon Cho](https://www.artstation.com/joncho3)
[Ajdin Talic](https://github.com/MagmarRager) | [Emil Carlsson](https://www.artstation.com/emilcarlsson)

### [Losing My Marbles](https://github.com/Llrac/losing-my-marbles)
![[Losing my marbles title.png]](https://github.com/emilxf-0/Portfolio/blob/main/Images/Losing%20my%20marbles%20title.png)

<details open>
 
 >**Developed:** 2022 Nov - 2023 Jan
>
>**Engine:** Unity
>
>**Language:** C#
>
>**Networking:** Firebase
>
>**Genre:** Couch co-op, Party game, Tactical puzzle
</details>

### Overview

Losing my marbles is a semi online, couch co-op game where you and your friends use your mobile phones to control your characters. Drawing inspiration from classic board game [RoboRally](https://boardgamegeek.com/boardgame/18/roborally) and party game favorite [Jackbox](https://www.jackboxgames.com/) we created a 2D puzzle/race game as a 7-week project at [Yrgo](https://www.yrgo.se).

### My contribution

**Networking**

I was responsible for making networking work. That meant setting up a database that Unity could talk to, create a lobby to ensure that we could run more than one game at a time (fairly important in terms of scalability), and create the controllers. 

**Project management** 

See note [at the top](#projects)

### Team: 
**Programmers** | **Artists** 
-------|-------
Me | [Tom Hammar](https://www.artstation.com/tomhammar)
[Max Petersson](https://github.com/Max-Petersson) | [Sofia Bjerned Hansson](https://www.artstation.com/sofiabjernedhansson)
[Carl Lycke](https://github.com/llrac) | [Jenny-Li Levin](https://www.artstation.com/jenny-lilevin) 
