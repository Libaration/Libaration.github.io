---
layout: post
title:      "My OO Ruby CLI Project"
date:       2020-08-15 02:25:57 -0400
permalink:  my_oo_ruby_cli_project
---

My CLI project is a gem that scrapes data from a few sources.
1. The official Riot games API located https://developer.riotgames.com/
2. A neat JSON list for all of the current League of Legends champions located http://ddragon.leagueoflegends.com/cdn/10.16.1/data/en_US/champion.json
3. OP.GG (Scraping)
4. http://www.asciiarts.net (Scraping

## The program allows users to pick through the current list of champions to find out more about an individual champion.
![](https://i.gyazo.com/7ec4fdce912bb089407a453fe5d15fbb.mp4http://)
## It allows the user to search for an account by username and pull up their most recent match history.
![](https://i.gyazo.com/1851aae5fb8bf2a0cd397cefcc48ed3f.mp4http://)

## It also allows the user to save accounts and switch between them from a list to quickly access details between them.

Originally I only wanted to use the Riot Games API, but to add some flair to when a user picks a champion I decided to scrape an ascii art generator for the champion the user picks as well.

```
  def img
    doc = LeagueInfo::Getdata.scrapeData("http://www.asciiarts.net/figlet.ajax.php?message=#{self.name}&font=isometric4.flf&html_mode=undefined&facebook_mode=undefined")
    image = doc.css("div#image").children[1].text
    puts "#{image}".light_blue ; puts "                                                   #{self.title}".light_blue
  end
```

I knew I was going to be working with a lot of data so I quickly researched for the best way to display that data through the terminal that led me to some incredible gems that did a lot of the leg work for displaying all the prompts in my application those gems were
```
  spec.add_dependency 'terminal-table', '~> 1.8'
  spec.add_dependency 'tty-progressbar', '~> 0.17.0'
  spec.add_dependency 'tty-prompt'
```

When a user first starts the program it loads the JSON file full of champions and creates objects from them.
Passing in the information as key value pairs.
```
  def self.load_champions
    champlist = LeagueInfo::Getdata.get('http://ddragon.leagueoflegends.com/cdn/10.16.1/data/en_US/champion.json')[:data]
    champlist.each do |_key, value|
      champion = self.new
      value.each_pair { |k, v| champion.send("#{k}=", v)}
    end
  end
```

This allowed me to easily list them using terminal table iterating through each object and calling upon those keys to return their values.

A problem I quickly ran into was when loading an accounts match history if I specified to load more than 10 games.
The progam only taking advantage of one thread at a time would hang and a user could easily think it has crashed or stop functioning. 
I used the gem tty-progressbar to show a visual representation of what the program was currently doing and so the user could see it was just loading a large amount of data and was in fact still running.

All of these gems I knew absolutely nothing about when I first began this project but after googling and looking through the docs endlessly I can say I'd be comfortable using them again if needed.
This project humbled me in many ways and pushed me to learn more to make the gem as good as I could. I'm excited to see what's in store for future projects and looking forward to being pushed out of my comfort zone to learn more!


