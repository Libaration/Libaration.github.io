---
layout: post
title:      "Rails Portfolio Project"
date:       2020-12-23 23:32:06 +0000
permalink:  rails_portfolio_project
---


I decided to create a music discovery platform for my rails project. Coming into Rails felt familar. The nesting was tough and the abstraction at some points became more cumbersome than helpful and I was thinking Sinatra was way easier.
After finishing my Rails project I can say I've grown to love and appreciate the level of abstraction Rails provided and it started making sense WHY we do things the Rails way. If you're trying to do something and rails just isn't allowing you an easy solution that probably isn't the rails way!

For this project I want to talk about a very neat gem I found to customize depending on color.
What I mean is for my project I have multiple Songs that can be shared and each song shows up on a users feed.
Displaying a boring grid with each song became stale. I found the solution in a gem called..
# Miro
Miro is a neat gem that extracts the dominant colors from an image and gives you the result in an array of HEX color values.
For each song's album art(the spotify API helped tremendously here) in my partial it would grab the dominant colors of its image and provide me with the hex color values which are most dominant. I extracted that into a neat little helper so its logic wouldn't be cluttering my view.
```
  def dominantcolor(song)
    Miro.options[:color_count] = 3
    colors = Miro::DominantColors.new(song)
    colors.to_hex
  end
```
Each song was nested in the div soup I created like so
```
<div class="songcontainer" style="background-color: <%= dominantcolor(post.song.image).last %>90;">
    <div class="albumcontainer">
      <img onmouseover="play('song_<%= post.song.id %>','<%= grab_background(post.song) %>')" onmouseout="stop_playing('song_<%= post.song.id %>')"  src="<%= asset_path('play_button.png') %>" id="playbtn">
      <img src="<%= post.song.image %>" class="albumart" id="song_img_<%= post.song.id %>">
    </div>
```

The important part of that code is the style="background-color" because you can see I'm calling that helper method which returns me an array of dominant hex color values, picking the last one out of the array and setting it to be that div's background color at an opacity of 90

This allowed me to dynamically have a different background color for each song that matched its album art without using javascript!

The final result is below
![](https://i.gyazo.com/897944677939f7e839225865812b4f6f.mp4http://)

I had a lot of fun with this project and am glad I completed it to truly appreciate Rails and it's simplicity
Excited for JS.
