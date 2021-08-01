---
layout: post
title: 'Final React Project'
date: 2020-08-01 00:05:56 -0400
permalink: final_react_project
---

Finally, I have reached the Final Project. I've grown to love React for how easy it makes reusing code and updating/rendering itself when changes are made with data.

## The Real Estate Auction react project.

For my last project I decided to try and go all out! I bit off more than I could chew in the alotted time frame but still I am happy with how far I've come!
My final project is a Real Estate auction website. Users can list homes for auction or bid on homes already listed. This is where I bit off more than I could chew. The bidding is imaginary as of now with no balances or way to pay for said bids, but the concept is all in place.
React made it easy to grab the data from my Rails backend and map over it sending each one to a resusable component to display that data.

## The issues...

So much data! Homes can have addresses, urls, bids, users who have bidded, bathrooms, bedrooms, auction ending dates and so much more attributes for each home. I found myself frequently going back and forth between coding my components and checking my actions/reducers just to remember all of these attributes! Working with so many moving parts often lead to silly typos, undefined values and runtime errors!

## The solution

#### Typescript!

Javascript is a dynamically typed language and some find that freeing and a joy to work with and while it can be, it also means a lot of errors you won't know about until runtime.
Typescript is a super set of Javascript which shows errors during compile time. You can see these in the terminal of your IDE which makes everything a lot easier. You don't find yourself trying to figure out where this undefined is coming from only to find out it was from a component you thought you had already finished on hours ago.
However, This does come at a price... Typescript is statically typed. You must define your types beforehand which could lead to a more verbose code and a bit more set up time, to me it felt less free but it forced me to really stop and think about what data I'm returning and where I'm sending it to.

## The Code

In Javascript for instance when I passed my state to props via redux before refactoring to typescript it looked like this

```javascript
const mapStateToProps = (state) => {
  return { homes: state.homes };
};
```

I'm returning an object to be passed to my props. My IDE has no knowledge over what is actually inside of state.homes or what type of data it is until I try to actually use it. If I wanted to to take 2 completely different values from my props like a Boolean and a String and try to add them together the compiler won't stop me and I wouldn't find out it was an issue until I actually ran that code.

In typescript we define those values beforehand so the compiler knows ahead of time everything it's dealing with for instance, this is my mapStateToProps in Typescript

```javascript
const mapStateToProps = (state: State) => {
  return {
    homes: {
      ...state.homes,
    },
  };
};
```

Almost identical what is the difference? well it's in this line here

```javascript
(state: State);
```

I'm passing that object to my props just as I did before but I'm telling the compiler what "types" are in that state I'm using to build my object.
I'm doing that by passing it something called an interface, i've named that interface state. It looks like this.

```javascript
interface State {
  homes: {
    homesList: Home[],
  };
}
```

and in this interface I'm telling it homesList contains an array of Homes interfaces which are defined like so

```javascript
export interface Home {
  id: number;
  address: string;
  bid: number;
  url: string;
  details: string;
  bathrooms: number;
  bedrooms: number;
  zoning: string;
  bids: Bid[];
  endDate: string;
  createdAt: string;
}
```

now I can use that home interface wherever I use that data and the compiler will know what data its recieving and how to work with it.
This gives us some pretty neat advantages like the predictive syntax in IDE's actually knowing what's inside of each home.
![img](https://i.gyazo.com/28fe602eb24ec67f771592a3f518f97d.png)

### in conclusion

I'm still learning to appreciate statically typing and the extra work / more verbose code it brings it's been and uphill battle but I've already been loving the benefits it brings. While refactoring my project to Typescript it became clear how much easier it would be to come into an already finished project that was in Typescript. You wouldn't need to hunt for these attributes or their types, which can be counter productive.

I still have A LOT to learn but I'm embracing the journey thus far!
