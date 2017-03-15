---
layout: post
title:  "Is There An Enumerator That Compares Nested Arrays?"
date:   2017-03-15 13:56:19 -0400
---


I just completed my Tic Tac Toe project with an unbeatable AI. I was able to get almost every part working how I wanted, except for one small bit of code in which I wanted an enumerator to iterate over nested arrays in a parent array and compare them to each other. Specifically, I wanted my AI to check whether the opponent player had a potential double. In other words, to check if there are two intersecting edges occupied by the opponent (and only the opponent).

![](http://i.imgur.com/QRRRhJG.png)

For example, in the above image, there is a potential double. Player "X" is occupying the top row and the right column, which intersect each other. If player "O" doesn't play a move somewhere in this intersection, "X" will have a guaranteed win.

To implement the logic of detecting potential doubles, I first programmed the AI to find all instances of lines that don't go through the center square and that contain exactly one opponent token and two blank spaces. 

```
opponent_lines = board.game.win_combos.select do |combo|
   a = []
   combo.each do |e|
      if e!=4
         a << board.cells[e]
      end
   end
   a.sort == [" ", " ", "#{board.game.opponent.token}"]
end
```

This would return a two-dimensional array containing win combination arrays occupied only by the opponent. 

```
opponent_lines
#=> [[0,1,2],[2,5,8]]
```

Next I wanted to compare the nested arrays to each other with `&` to check if they intersected at any position on the Tic Tac Toe board. 

```
opponent_lines.inject(:&)
#=> [2]
```

This works fine if there is only one intersection, but it breaks down if your opponent has two possible doubles with two intersecting lines. 

![](http://i.imgur.com/5GQDT4v.png)

In this case `opponent_lines` is an array with three nested arrays.

```
opponent_lines
#=> [[6,7,8],[0,3,6],[2,5,8]]
```

The inject method I was using no longer returned a useful value.

```
opponent_lines.inject(:&)
#=> []
```

Then I saw this approach somewhere on Stack Overflow: `opponent_lines.inject(&:&)` and gave it a shot. 

```
opponent_lines.inject(&:&)
#=> []
```

Clearly, this too was useless for my purposes. Apparently it only returns a value found in all three nested arrays. But I needed to see if two out of three nested arrays intersected.

Perhaps iteration was the answer. But I only know how to iterate over an array one element at a time. I tried to find an existing enumerator that could compare multiple nested arrays in a single parent array at once. The closest thing I found was the [`each_cons(n)`](https://ruby-doc.org/core-2.4.0/Enumerable.html#method-i-each_cons) Enumerable. The "cons" is short for consecutive, and the method operates on consecutive elements in an array, where the number of consecutive elements is specified by `n`. 

```
intersects = []
opponent_lines.each_cons(2) {|e| intersects << e.inject(:&)}
intersects
#=> [[6],[]]
```

This method also doesn't work, because it only compares consecutive arrays in `opponent_lines`. Remember, in the second scenario, `opponent_lines` has three nested arrays:

```
opponent_lines
#=> [[6,7,8],[0,3,6],[2,5,8]]
```

With the `each_cons(n)` method `opponent_lines[0]` and `opponent_lines[2]` are not compared. I needed a method that returns `[[6],[8]]`.

Additional searching produced no existing enumerator that could perform the task I wanted done. So, I changed my approach. I realized I could just flatten the entire `opponent_lines` array and find all elements that appeared more than once, and these would indicate intersection points.

```
f = opponent_lines.flatten
intersects = f.select { |e| f.count(e) > 1}.uniq

intersects
#=> [6,8]
```

Next, I compared the positions returned by `intersects` with available positions on the board, and told the AI to place its token in the appropriate square.

In the end, the code I chose to use worked just fine, and seems abstract and eloquent enough. But I still wonder if there is an enumerator out there that can compare nested arrays to one another?

