
## Downloading an entire album from Photobucket
I found the old code for the worm that I helped to create, and wanted to rehost it. Back in the day though,
I hosted the code myself and hosted all the images at photobucket (this was around 2007), because hosting images
was too expensive for me.

It's twelve years later now, and photobucket doesn't look so good anymore. Coupled with my recent interest of getting off
as many platforms as I can, I decided to host the images myself too. I have a selfhosted solution now anyways, so transaction 
and hosting costs are boiled down to electricity bills. (You can't hotlink images hosted on photobucket anymore, 
so using it as separate storage wouldn't have worked anyway).

So, I found myself in the situation where I wanted to download an entire album from photobucket. 
When I was not logged in, there was a nice "download album" button, but if you click on it, it asks
you to log in, and when you do, the download button disappears :)

A quick google later, it turns out that it's not been possible to download an entire album in one go since forever, and apparently, 
you have to pay for such services. Well, ok.

I had faith in the internet though, and I found the following site quickly enough: 
[Download all your photobucket images in bulk via CLI](https://gist.github.com/philipjewell/a9e1eae2d999a2529a08c15b06deb13d).


As he describes there, you can select all the images in an album (by selecting one and then clicking select all - that selects 
all images in the album, even those on different pages). Then, you can click on link, and copy the direct link text. You'll get something like
this then (I had to right click the highlighted text and choose "select all" for all of it to be selected btw):

```
http://i240.photobucket.com/albums/ff67/wormmanager/ogame/w4.jpg    
http://i240.photobucket.com/albums/ff67/wormmanager/ogame/w3.jpg    
http://i240.photobucket.com/albums/ff67/wormmanager/ogame/w2.jpg    
http://i240.photobucket.com/albums/ff67/wormmanager/ogame/w1.jpg
```

I saved that to a text file `pd.txt`, and in the same folder, I ran the following command:

```bash
cat pb.txt | while read file; do curl -O --referer "https://i240.photobucket.com/" ${file}; done
```

I found out that the referer had to be the same as the image location, using `"http://s.photobucket.com/"` like in the tutorial
didn't work for me.

So yeah, I managed to download about 150 images relatively quickly. 

### For those on Windows
Take a look at the comments under his post: [Download all your photobucket images in bulk via CLI](https://gist.github.com/philipjewell/a9e1eae2d999a2529a08c15b06deb13d), there are
some scripts there for Powershell, which you can run on Windows. 

