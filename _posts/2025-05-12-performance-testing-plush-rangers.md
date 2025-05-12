---
title: Performance Testing Plush Rangers
layout: post
date:   2025-05-12
excerpt: For the most recent update in [Plush Rangers](https://store.steampowered.com/app/3593330/Plush_Rangers/), I focused on improving the performance of the game. I wanted to share some tips I learned while going through the process that might help others that are looking to optimize code. 
---

For the most recent update in [Plush Rangers](https://store.steampowered.com/app/3593330/Plush_Rangers/), I focused on improving the performance of the game. I wanted to share some tips I learned while going through the process that might help others that are looking to optimize code. 

I didn’t use any “tricks” to improve the performance, it came down to using a consistent, methodical approach to understanding what was happening and finding improvements. If you have any suggestions on performance testing, leave them in the comments!

### Set a target benchmark

**You need to know when optimizations are done.**

My release performance target for this game (at release) is supporting ~500+ enemies active with corresponding effects, projectiles, and other tidbits on the screen at the same time while playing on a Steam Deck.

### Set a goal for today

**You don’t need perfect today, you just need to stay on course to your final goal.**

Even after this round of optimizations, I’m not 100% of the way to my goal yet. I’m ok with this. I know that there will be many things that will change in the project between now and release. For iterative optimizations I’m trying to stay in contact with my goal so that as the game reaches it’s final stages the last rounds of optimization are easier to achieve.

### Build a test bed that breaks the performance in your game

**Make a test that is 2-5x what your target goal is to break the performance of the game and find issues at scale.**

Testing in normal gameplay will introduce a lot of variables and make it difficult to compare changes. In order to test your performance code changes methodically, you need a consistent comparison. Create a test environment that is as repeatable as possible that pushes beyond your target goal.

### Profile and Diagnose

**The profiler tells you where to look, but not why something is slow.**

When I profiled my test bed I found that drawing was taking ~45% and enemy step was taking ~45%. That was interesting. In normal operations enemy movement was like 5% of the time and drawing was 60%+. I was dealing with two different kinds of problems. 

1. The enemy movement was a scalability problem. This points to structural inefficiencies. 
2. The drawing scaled well but any optimizations in a performance heavy routine will help.

### Comment out code to find the problematic areas

Before I started making more changes, I need more information. What was exactly causing things to slow down? Was it loops, a specific routine, bad logic? To find the real problem areas and figure out how code was interacting, I commented out as much code as I could and still run the test. Then I reintroduced a small bit of a code at a time. 

For example in my drawing routine, I commented out all the drawing and then just reintroduced constructing a matrix. I could see how it was performing and figure out if there was any wasted energy in that small section of code and test that I could improve that. 

### Solving Scalability Problems

For my enemy step event code there were a few things that was making my code slow:

1. Collision detection: Enemies were checking frequently whether they were bumping into obstacles or edges of the map. This isn’t a platformer with really tight areas, I could get away with simulating it more and doing it less. I solved this by using alarms to only check for collisions periodically. These alarm rates are configurable per object, so I can always fine tune specific behavior. 
2. Moving around obstacles: On top of that, there was a lot of attempts to try and move around obstacles (check x + this, y + that, etc…) Instead of checking lots of areas every frame I set up a variable to search a different direction the next frame. This stops the enemy for a tick, and then it will search next frame. In the course of gameplay, you cannot notice the delay but it saves a ton of cycles per frame. 
3. Dealing Damage: So, I made a really cool ability system in my game to allow adding different kinds of attacks and skills to characters in the game. It’s really modular and allows a lot of customization. It also adds a bit of overhead. That overhead is negligible on the interesting characters like the player, your friends, or bosses, but it eats up a ton of time doing basic stuff for enemies. So I removed that for the basic enemies and streamlined their code. Lesson here: Don’t become attached to your code when it gets in your way. Sometimes it’s best to just do it instead of making it pretty. 

### Making the fast, faster

Because my game is drawn using a perspective camera and billboarded sprites, relying on the traditional Gamemaker drawing system wasn’t an option. All my drawing code goes through a centralized camera that loops through the appropriate objects to draw in the optimal order. (This is actually an interesting and easy to implement system). At times though, it was taking up too much energy I came across a few items to help improve performance.

1. I found dead code in this routine. It was from previous iterations of drawing shadows that I had slowly drifted away from. Deleting out a few ifs and math makes a difference when it happens 1000+ times a frame.
2. I was not using some libraries correctly. I have a 3D particle library I’m using that works great, but the way I configured it led to slow downs after it had been running for a long time. Once I dug into the code and understood better how it worked, I modified my usage of the library to be better optimized.
3. The graphics functions, texture swaps, vertex batches were not that critical to performance. I did do some optimizations in organizing my texture pages to better utilize the process and I think it helped some.
4. Consistency helps performance. The majority of my objects use the same shader, the same matrix behaviors, the same sprite behaviors. There are a few configuration options but these are not checked against but just passed into the shader as values. There are some objects to draw that cannot follow this like particle systems, but for my basic sprites they all work the exact same way. This eliminates lots of checks, it eliminates calling custom code for each object. 

Here’s a little sample video of a busy moment in the game after working through these tests. This is actually still in VM build and a full release build would perform even better.


<iframe width="560" height="315" src="https://www.youtube.com/embed/M29hFzhN6Jw?si=6Ybo8Rw72j_uGLXa" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### About this game

**Plush Rangers** is a fast-paced auto battler where you assemble a team of **Plushie Friends** to take on quirky mutated enemies and objects. Explore the many biomes of **Cosmic Park Swirlstone** and restore **Camp Cloudburst**!

[Wishlist Plush Rangers on Steam](https://store.steampowered.com/app/3593330/Plush_Rangers/)