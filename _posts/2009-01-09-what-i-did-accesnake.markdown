---
layout: post
title:  "AcceSnake : Learn Symbian OS through a great Snake game development"
date:   2009-01-09 12:00:00
tags: symbian python application game english whatidid
---
What I did from 2008 to 2009? I was bored so I learned **Python** and **Symbian OS** development and created a popular Snake game using the accelerometer.

- [Why?][1]
- [AcceSnake : A Snake Game using the accelerometer][2]
- [End of the project][3]
- [Links][4]

    [1]: #why-1
    [2]: #accesnake-2
    [3]: #end-3
    [4]: #links-4

<a name="why-1" />

Why?
----

During summer 2008, I worked 1 month in England (I'm French), mentoring teenagers during their English class trip. Main of my work was to wait during their English lessons. At that time, I didn't have a computer with me, but a great **Nokia N95 8Go** under **Symbian v5**. This awesome smartphone had no touchscreen but **a nice T9 keyboard**, plus many navigation buttons (11 keys, with the D-pad). It's **2,8 inches screen** (240x320px) was pretty big at this time. And the great thing was all the new sensors we were not familiar with. GPS, 5MP Camera, TvOutput, a fast 332MHz Dual CPU and it's proper 3D Graphics HW Accelerator GPU! But the awesomeness was **the accelerometer**. It was so brand new that there was no released driver for... and so no game, no application, nothing using it. But the specs said it was there!

<img src="{{ site.url }}/assets/images/accesnake_n958gb.jpg" alt="N95 8GB" style="width: 400px; margin-left: auto; margin-right: auto"/>

So there was this game, **3D snake**, a great snake game reborn. I spent all my time on it and finished it once, twice then was bored. I was even more bored because the game was great but it didn't use 2 things I wanted in it: an accelerometer control, and a 360° liberty of move.

<img src="{{ site.url }}/assets/images/accesnake_snake3d.jpg" alt="Snake 3D" style="width: 200px; margin-left: auto; margin-right: auto"/>

Then what to do when you want something you don't have? You make it! But *I didn't have computer with me*, and no data most of the time. So I texted my brother who know well computer science and asked what can I do. *"Learn Python, and find an interpreter."*. So I did. I found almost immediately an awesome small community of Symbian developers, **an official (!!!) python interpreter and API made by Nokia**, and also, and that's the great part, **a leaked accelerometer driver** with it's API made by the community. Then all I needed was a notepad (easy part, of course).

<a name="accesnake-2" />

AcceSnake : A Snake Game using the accelerometer
------------------------------------------------

<img src="{{ site.url }}/assets/images/accesnake_logo.png" alt="AcceSnake logo" style="width: 200px; margin-left: auto; margin-right: auto"/>

{: .center }
[Source code and download page](https://code.google.com/p/accesnake/downloads/list)

{: .center }
[Official page of the game](http://cmoatoto.fr/accesnake.html)

> AcceSnake is a "Snake-like game" using the accelerometer of your phone.
>
> Developed in Python, this game has been entirely coded with a text-editor, on a N95 8Go. I never used a computer to make it.
>
> For the first time in a snake game, you can move in any direction, slow down, and accelerate just by moving the phone. You can cross your tail but don't touch the wall or you will lose a life.
>
> If you don't have an accelerometer, or if you prefer to use the keyboard, just choose the right option.

I will not explain everything about the development. It's not that interesting now. I did the first version during the summer, released it on **october 2008** and release some new version **until january 2009**. I did a lot of communication on game forums so I had quite a success. 

My last version was downloaded **more than 50,000 times** on the [official download page](https://code.google.com/p/accesnake/downloads/list) and **more than 200,000 times** on various specialized website.

Here is some notes about the development:

---------
Free Open Source game available on [code.google](https://code.google.com/p/accesnake) and [CmoaToto.fr](cmoatoto.fr/accesnake).

- The goal was to actually learn mobile development
- UI development (menus, dialog boxes, graphics...)
- Management of UI and life of the game (gameLoop).
- Plugin management (for accelerometer use).
- Event management of the accelerometer.
- Managing the distribution and advertising.
- Internationalization (assisted by community for regionalization).

<img src="{{ site.url }}/assets/images/accesnake_screenshot.png" alt="AcceSnake logo" style="width: 240px; margin-left: auto; margin-right: auto"/>

<a name="end-3" />

End of the project
------------------

During february 2009, I rewrote all the code, using better Objects, creating **multiplayer** gamemode, special funny things etc... **I had an awesome v3 in my hands**. *Then I broke my phone*.

\*sigh\*

So I bought a new one. a **Nokia N96**. Without T9 but **a complete landscape keyboard, a touchscreen, no D-pad, and with Symbian v6**. I had to rewrite everything to make it work.

So I started...

Then I stopped. *(but I started again, on Android, in december 2011 ;) )*

<a name="links-4" />

Links
-----

[Official website](http://cmoatoto.fr/accesnake.html)

- [presentation](http://cmoatoto.fr/presentation.html)
- [screenshots](http://cmoatoto.fr/screenshot.html)
- [installation](http://cmoatoto.fr/installation.html)
- [links & help](http://cmoatoto.fr/links.html)
- [high scores](http://cmoatoto.fr/highscore.php)

[Google Code](https://code.google.com/p/accesnake)
