---
layout: post
title:      "Recreating 2005 Myspace with Sinatra"
date:       2020-09-29 21:52:30 +0000
permalink:  recreating_2005_myspace_with_sinatra
---


For my Sinatra final project, I decided to go back to where it all began. If anyone used Myspace in it's prime you'll know it allowed you to customize your profile. That's where my initial curiosity for coding began so in a weird way I found myself in a full 360 now coding the thing that got me interested in the basics of coding???

The recreation is a CRUD representation of Myspace how it was originally in the year 2005.

![](https://i.gyazo.com/633ac419b9a7680b486fd301efc58902.jpg)

### It features a lot of the original websites functionality including
1. User Signup
2. Profile Comments
3. Messages and message notifcation
4. Friend requests and friend request notifcations
5. Custom CSS for user profiles (Old myspace layout site codes work here too! Although a bit buggy sometimes)
6. Profile pictures and user photo albums

Notably it's missing the ability to post blogs on your profile because the project requirements said we weren't allowed to do those since we built those in previous labs and also bulletins/home page for reasons I'll get into later.
After I figured out what I'd like to do for my project I went on the [Way Back Machine ](https://archive.org/) typed Myspace.com and grabbed the source for a 2005 screenshot of it to give me my inital template. Since CSS/HTML weren't the main concerns for this project it worked out since I'd mostly be doing backend and all of the front was kind of already handled in a way. There were no screenshots to grab for when a user is logged in so I had to improvise since I could not see the source for a logged in user's home page in the year 2005. I wound up just adding that to a users profile if they're logged in.
This project pushed me and was incredibly challenging but it was fun living in nostalgia whilst coding this even if there were some issues I ran into.....


### The problem
The initial setup to get everything running was relatively simple a User can have many photos and photos belong to a User
Where it started to get a little heretic was building the relationship for my User model being friends with another instance of my User model.
The first thing I attempted was to add a has_many relationship to my User model
> class User < ActiveRecord::Base
> 
> `has_many :friendships`
> 
> `has_many :friends, through: :friendships`
> 
> end


and then create a jointable called Friendships which would hold a user_id and a friend_id this join table would act as the link between friends for instance Tom's ID was 1 because he is the first User to ever sign up. My ID was 2, and by giving my User model the previously mentioned has_many through relationship I could now call a .friends method *(that was generously provided by active record)* on a User which would return to an array containing the friend_id where every user_id of my friendship join table matches the id of the User the method is being called on because my friendship table BELONGS to a User
The reason I have to specify `class_name: User` is because if I don't active record won't know how to tell the two apart. Although a User and a Friend are both from the same model active record doesn't know that unless we give it that information.

```
class Friendship < ActiveRecord::Base
  belongs_to :user
	belongs_to :friend, class_name: 'User'
end

```


For instance Tom is automatically friends with everyone on Myspace so me becoming his friend would look like this.

> @tom = User.find(1) *#this finds the user who has an ID of 1 in this case it's Tom!*
> 
> @cristian = User.find(2) *#this finds my profile I have an ID of 2*
> 
> `@tom.friends << @cristian`
> 
> `@cristian.friends << @tom`

And just like that, I have shoveled myself into Toms friends list and I am now his friend, and because I want the friendship to be reciprocal I have also shoveled Tom into MY friends list.
Great! Everyones friends now. It's all as it should be right?
Well, that's what I thought until I realized you can't just add yourself to someones friends list without their consent! That's so invasive right? Maybe Tom is incredibly irritated I ripped off his source he most definitely wanted to reject my offer to be friends.
### The Solution
To remedy this issue I thought how can I have a place where friendships stay until they are approved or denied?
I then added a boolean value to my friendship join table named accepted at this point my friendship table looks like this.
```
 create_table "friendships", force: :cascade do |t|
    t.integer "user_id"
    t.integer "friend_id"
    t.boolean "accepted",  default: false, null: false
  end
```

It's important that the boolean can not be null and defaults to false since when we send a friend request it isn't accepted right away.

I could now create a relationship in my User table for when that boolean is either true or false
```
class User < ActiveRecord::Base
has_many :friendships, -> { where(accepted: true)}
has_many :friends, through: :friendships
has_many :pending_friendships, -> { where(accepted: false)}
has_many :pending_friends, through: :pending_friendships, :source => :friend
```

Now if I wanted to be friends with Tom the code would look like this
> ```
>  @tom = User.find(1)
>  @cristian = User.find(2)
>  @tom.pending_friends << @cristian
> ```
> 
I'm shoveling myself into Toms PENDING friends list, and the process of accepting that request would look like this 

```
@Tom.pending_friends.delete(@cristian)
@cristian.friends << @tom
@tom.friends << @cristian
```

I'd remove @cristian from Tom's pending users and add @cristian to Toms FRIENDSLIST. There's probably an easier way to go about this but this is the solution I came upon after lots of googling.

There are PLENTY of other issues I ran into trying to recreate 2005Myspace but I learned so much about how relationships work in ActiveRecord and how abstract they can be. I'm glad I was forced to really dive deep and graph these relationships out because before I ran into this issue and the many after it I wasn't very confident in building relationships.
I had a lot of fun with this project and can't wait to work with Rails and compare the similarites I've learned here!

