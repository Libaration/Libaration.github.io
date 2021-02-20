---
layout: post
title:      "Instagram Feed Designer JS Project"
date:       2021-02-20 13:22:14 -0500
permalink:  instagram_feed_designer_js_project
---


We have finally arrived at JS, something I feared from my childhood dabbling in web design. 
Whilst all my friends spoke of this new cool jquery libary I simply did not take interest because Javascript syntax looked so unruly.
So, what have I learned after completing my JS project? Well... javascript syntax is still in my eyes verbose and unruly, BUT with a firm grasp on why JS does things this way and why javascript has some... interesting edge cases to say the least I no longer have that fear of JS that I once did have growing up. In fact diving into it I found it can be similar to C# in ways! So, translating some of that experience over has helped me not get lost in the curly bracket madness.


For my final JS project we were tasked to create a backend API in rails to fetch JSON data from. I really hate working with data you submit yourself because it doesn't feel quite real to me. At least in the sense that most API's your fetching information from already have useful information you can build your front end with. 
So, for this project when a user submits I chose to scrape instagram for the rest of the users information thus filling up my backend API with a ton of information I could use to build my front end out with.

## The Instagram Feed Designer
As a photogapher I absolutely love planning out my instagram feed. It's more important than just posting what you feel like you want to post the day. Carefully planning out your feed so each image cordinates with the others on your profile makes your instagram have a full cohesive image, unfortunately instagram doesn't have this feature of pre-post planning built in and the many 3rd party applications require a subscription so I decided to build one for myself.

## Rails Backend
For my backend whenever a user is submitted it creates the user in my rails backend and then scrapes for their last 12 instagram photos adding those to the Photo model of my database and creating the relation that those photos belong to the user. This allowed me to quickly build the typical "Instagram Grid" on the front end.
In my User model I set a :before_save action to call those scraping methods.
```
class User < ApplicationRecord
    has_many :photos
    before_save :scrape_avatar, :scrape_name
```
and in those functions the scrape grabbed the information and set it's own properties to the information recieved so I had an avatar image and the users name
```
def scrape_avatar
        self.image = load_site.css(".profile-avatar img").attr('src').text
    end
def scrape_name
        self.name = load_site.css("h2").text
end
	```
	

### Great!
With that information I had a baseline to build out my app. I used that information to create a bio for the user and an icon on the front end like this, it lives in a side panel that is revealed on user submit.
![](https://i.gyazo.com/b14ae74fbec67d14fea60f8f84d8d21b.pnghttp://)




The backend function `scrape_photos` in the User model scrapes the most recent IG images and creates a relation between the current instance of the user and the newly created photos it looks like this.
```
    def scrape_photos
            images = load_site.xpath('//div[@class="content box-photos-wrapper"]').css("li").css("img")
            images.reverse.each do |img|
                if Photo.exists?(['url LIKE ?', "#{img.attr('src')}"])
                    next
                end
                    url = img.attr('src')
                    caption = img.attr('alt')
                    self.photos.create(caption: caption, url: url)
                end
    end
```
Basically if the image already exists go next. I don't want to add the entire feed again into my database causing duplicates if the same user is loaded twice otherwise lets scoop that information we need into the Photo's model.

With that information I could build out the "Instagram Grid" that also reveals on user submit.
![](https://i.gyazo.com/bce24977fa331ce8569b934432183e0d.jpghttp://)

With most of the information I wanted out of the backend done I now had to move onto the exciting designing phase of the front end

## Javascript and the Frontend Design
For this I wanted it to mimic how instagrams photos are displayed since you want to be able to plan your photo posts ahead of time and see what they'll look like on your feed without actually making the post on instagram.
I created a simple grid layout with CSS to achieve that
The photoGrid would live in a 3x3 container allowing them to grow and stay in the same general layout as users add more photos.

In JS I added an event listener to listen for when a user is submitted
```
userForm.addEventListener("submit", loadUser);
```
It then calls a function called loadUser which would fetch to my rails backend to grab this information needed that we created before to build the layout
```
const loadUser = async (e) => {
    e.preventDefault();
    const igHandle = document.getElementById('ig-handle');
    let user = await apiService.createOrFindUser(igHandle.value.replace('@',"").toLowerCase());
    const {id, username, image, name} = user;
    user = new User(id,username,image,name);
    user.displayUserInfo();
    Photo.clearPhotos();
    user.displayPhotos();
    let addPhotoForm = document.querySelector('form.addpopupform')
    let clonedAddPhotoForm = addPhotoForm.cloneNode(true);
    addPhotoForm.parentNode.replaceChild(clonedAddPhotoForm, addPhotoForm);
    clonedAddPhotoForm.addEventListener("submit", user.addPhoto.bind(user))
    Animations.revealFeed(e);
}
```

First we prevent default so the page doesn't reload, then we grab the information from textbox that contains the user they want to submit.
We then make a fetch call to a ES6 class that holds all relative API calls I will make throughout my application. Seperating these concerns made the code a lot easier to read. Inside of the class specfically it's calling my createOrFindUser function which basically just either inserts a new user into my database or pulls up the one that already exists.
```
async createOrFindUser(handle) {
        const response = await fetch(`${this.baseURL}/users`, {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "Accept": "application/json"
            },
            body: JSON.stringify ({
                user: {username: handle}
            })
        })
    return response.json()
    }
```
In this code my baseUrl is set to localhost:3000 in a constructor for the class but the important part of this code is we're making a POST request to the rails backend. That will land on our CREATE route in the rails backend. Which will give us the JSON data response we need to build out the front end. It looks like this
##### RAILS CODE.
```
    def create
      user = User.find_or_initialize_by(userparams)
      if user.save
        user.scrape_photos
        render json: user
        else
        render json: {Status: "Failure"}
      end
    end
```

This renders out a JSON file like so 
```
{
  "id": 15,
  "username": "madisonbeer",
  "image": "https://instagram.flwo4-1.fna.fbcdn.net/v/t51.2885-19/s150x150/118521642_2687295488188699_1061980578101845113_n.jpg?_nc_ht=instagram.flwo4-1.fna.fbcdn.net&_nc_ohc=QUZzOXk9STsAX95Q4md&tp=1&oh=aabf14e19326ce341330c8c7c2fabb08&oe=6055F0F8",
  "created_at": "2021-02-18T21:19:47.719Z",
  "updated_at": "2021-02-18T21:19:47.719Z",
  "name": "Madison Beer",
  "photos": [ ... ]
```

I replaced the photos key with ... but inside of the photos key is an array with all of the Photos that belong to that user.

Okay back to the Javascript. Now we've reached lines
```
const {id, username, image, name} = user;
    user = new User(id,username,image,name);
    user.displayUserInfo();
    Photo.clearPhotos();
    user.displayPhotos();
```

We use ES6 to deconstruct the user json response we recieved and set the variables
Then a new ES6 class which constructs a user is created
That class holds functions relating to to that specific user like displaying its Photos or User Info
We can now call those instance functions because we have created the new User with the information we recieved.
Here's what the user function looks like to display those photos.
#### APISERVICE CLASS
```
async displayPhotos() {
        const userObject = await apiService.grabPhotos(this);
        for(let photo of userObject.photos){
                let {id, caption,url} = photo;
                photo = new Photo(id,caption,url)
                photo.displayPhoto();
        }
    }
```
We do the same as we did for User but this time create a new photo
and with each Photo we create a new ES6 Class for Photo's which has functions of it's own as wel!
So we can now display that photo which will render everything to the DOM showing us all users photos.
#### Photo Class
```
displayPhoto() {
        const overlay = document.createElement('div')
        const destroyBtnDiv = document.createElement('div');
        const destroyBtnImg = document.createElement('input');
        const overlayContainer = document.createElement('div')
        destroyBtnDiv.classList.add('destroyBtn')
        destroyBtnImg.id = `destroy#${this.id}`
        destroyBtnImg.setAttribute('type', 'image')
        destroyBtnImg.setAttribute('src', 'images/close.png');
        destroyBtnDiv.appendChild(destroyBtnImg)
        destroyBtnImg.addEventListener('click', this.destroyPhoto.bind(this))
        overlayContainer.classList.add('container')
        overlay.classList.add('overlay')
        overlay.innerHTML = `${this.caption}`
        overlay.appendChild(destroyBtnDiv)
        overlayContainer.appendChild(overlay)
        const photoContainer = document.querySelector('#photoContainer')
        const photoRow = document.createElement('div');
        photoRow.classList.add('row')
        const photoImage = document.createElement('img')
        photoImage.classList.add('feedImg')
        photoImage.src = this.url;
        const caption = document.createElement('h3')
        caption.innerText = this.caption
        photoRow.appendChild(photoImage)
        overlayContainer.appendChild(photoRow)
        photoContainer.insertBefore(overlayContainer, photoContainer.firstChild);
    }
```

There's a lot more in terms of JS that was done for this project but I think these are the most important parts such as animations and making each photo draggable!
Here's a final image of how it looked when it was finished.

![](https://i.gyazo.com/7fed806995f913fa3d3e2ceaeb5fd1f6.mp4http://)
