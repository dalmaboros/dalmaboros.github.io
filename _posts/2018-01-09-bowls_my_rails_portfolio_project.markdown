---
layout: post
title:      "Bowls, My Rails Portfolio Project"
date:       2018-01-09 16:49:34 +0000
permalink:  bowls_my_rails_portfolio_project
---


The process of creating my Rails Portfolio project was drawn out and full of mistakes. It started with an idea. I wanted to have a bowl in real life, fill it with scraps of paper, each with an activity written on it. In my downtime I could pull a random scrap from the bowl, and do the activity written on it, filling my free time with productive things. Bowls is a digitized version of that idea.

I find myself making hand-written notes to capture all the pieces of a web app floating around in my brain. There are so many moving parts to a web app that it really helps make space in my mind for next steps in the process. Here is my first note (complete with irrelevant doodles):

<img src="https://i.imgur.com/OMS0IGN.jpg" width="500px">

For me, the most logical starting point is mapping out the models and their associations. In this case, it's pretty uncomplicated with only three models: `Bowl`, `Scrap`, and `User`. A User has many `Bowls` and `Scraps`. And `Bowls` and `Scraps` have a many-to-many relationship with each other, expressed through a `Bowl_Scrap` model. 

Next, I jump right to the views because I want to think about what the app is ultimately going to do and how it's going to be, and then code the blank inbetween parts. So I made the routes and actions as I went, relying heavily the Rails exceptions the app threw at me along the way. 

Again, I made hand-written diagrams to map out how I wanted Bowls to behave:

<img src="https://i.imgur.com/UCyQyuQ.jpg" width="500px">

Once I had all the views and CRUD actions I needed solidified, I made a function that lets you pull a random scrap from a bowl. I also made a scope method that shows you all the scraps that have no bowl. I also implemented OmniAuth with Devise, which was a lot more difficult than I expected, but I managed with a considerable amount of help from Google and Stack Exchange. 

The finishing touch was refactoring the code a little bit, and DRYing it up. Then I recorded a video walkthrough showing how to use the app, which actually revealed a bunch of little bugs that needed fixing. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/o_QCkHA5JUw" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

**Some things I noticed:**

I didn't implement every feature I wanted (like multi-strategy OmniAuth), because some things were just a little too involved, and I needed to strike a balance between functional enough and spending too much time on functionality that's not totally crucial.

Next time I would save the styling for the end of the project, once the functionality of the app is finished, because it feels a little too scattered to work on both simultaneously.

I will always always make hand-written notes and diagrams!

This stuff is really hard, and that's okay!
