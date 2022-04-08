# Reddit-Place-Cropped-Timelapse

While there are many great timelapses for r/place, I only worked on a small [21x15 grid](https://velkog.com/place). So in a full picture timelapse, the art is nearly invisible. What I wanted was a upscaled and cropped timelapse showing the surrounding area of the art I worked on. I tried taking existing timelapses and do this in video editors, had trouble precisely altering the video how I wanted so I did it manually using [ffmpeg](https://ffmpeg.org/). This allowed me to crop to exact cordinates and upscale the image to 4K without any blurring.

I ended up liking the results and figured other communities might want to create their own versions. For now, this repo just contains the commnands I used to make the video. I was considering making a program to automate the entire process, but it's only a few simple commands and I'm not sure if there's any demand for it - so this is all for now. These instructions should work for both macOS and Linux. For Windows you can use WSL, otherwise I believe everything should work except for step 3 where the for loop will need to be rewritten.

Here's a sample video I made using the following commands, or [watch on youtube](https://www.youtube.com/watch?v=pG87i8e5FWA).

https://user-images.githubusercontent.com/60084050/161900281-57263b74-2884-46f6-8d0b-680e538e90b7.mp4

### 1. Download ffmpeg
Not much to say here - download ffmpeg however makes sense for your OS. You can [download the executables directly](https://www.ffmpeg.org/download.html), but it probably makes more sense to use your system's package manager.

### 2. Download Image History
Using the [rplace.space](https://rplace.space/combined/) archive created by [rission](https://github.com/rissson) and [ProstoSanja](https://github.com/ProstoSanja) we're going to download the entire history or r/place broken down into 30 second intervals. In total this is 15 GBs, so make sure you've got enough storage space available, and depending on your internet speed can take quite a bit of time:

```
wget -r -np -R="index.html" https://rplace.space/combined/
```

This will create a folder in your current directory called `rplace.space` containing another folder `combined` which is where every image will be downloaded in full resolution.

### 3. Crop and Upscale All Images
Next step is to edit the image in two parts:
1. Crop it so we only deal with the portion of the image we care about
2. Upscale the image to create sharp pixelated images without any blurriness in typical video upscaling.

To break down the command a bit:
* We create a new folder `upscaledcropped` under `rplace.space` where we will write all of the cropped and upscaled images to.
* Create variable `frame_num` used for counting each image we process and labeing the outputs. Likely not entirely necessary, but I was having difficulty running ffmpeg without this step.
* For every single picture we downloaded we want to do the following image processing command
* The image processing command will crop and upscale the photos, then save the pictures named sequentially by frame number. The following values you'll want to edit are the following:
  * **`240:135`** - this is the width:height of the area of interest. 
  * **`589:338`** - this is the starting x:y value where your area of interest begins (the upper-left corner).
  * **`3840x2160`** - this is the output resolution of the image. I wanted a really high quality video so I opted for 4K. If you don't care about having as high of quality you can save yourself some time and space and lower this image. To make sure the upscaling works well, I would make sure this dimension is multiple of your input dimensions (i.e. 3840x2160 is 16 times 240x135).
  
This is to be run as one singular command:

```
mkdir rplace.space/upscaledcropped; \
frame_num=0; \
for filename in rplace.space/combined/*.png; do \
    ffmpeg -i $filename -filter:v "crop=240:135:589:338,scale=3840x2160:flags=neighbor" \
    rplace.space/upscaledcropped/$frame_num.png; frame_num=$(( frame_num + 1)); done;
```

### 4. Render Images Into Video
The next step takes all the cropped/upscaled images and combines them into a single video entitled timelapse.mp4
* **`-framerate 60`** - sets the framerate to 60 frames per second. This results in a ~3 minute video.
* **`-crf 0`** - turns off all video compression. This is very important to having a crisp and clear video.

```
ffmpeg -framerate 60 -i rplace.space/upscaledcropped/%d.png -c:v libx264 -crf 0 timelapse.mp4
```

**OR**

I wanted to have the final frame of the video extend for a bit longer so I added a tpad video filter to the previous command to extend the video by 5 seconds. If you don't care about this, you can use the previous command.
* If you want shorter/longer of extension than 5 seconds, adjust the `stop_duration=5` value.

```
ffmpeg -framerate 60 -i rplace.space/upscaledcropped/%d.png -vf tpad=stop_mode=clone:stop_duration=5 -c:v libx264 -crf 0 timelapse.mp4
```

### 5. (Optional) Add Audio
To really put the cherry on top, I added some audio. Although very over the top, I used [Diva Dance from The Fifth Element](https://www.youtube.com/watch?v=jDJ4AaTPqfw) - definitely over the top, but that's the feel I was going for - plus it synced up nicely.

Use the following command, where:
* **`audio.mp3`** - is your audio file
* **`-shortest`** - assumes your audio is longer than the video, and will trim the audio. If not remove this from the command.

```
ffmpeg -i intermediate_extended.mp4 -i audio.mp3 -map 0:v -map 1:a -c:v copy -shortest timelapse_with_audio.mp4
```

### 6. Contact
If you have any questions, suggestions, etc. feel free to contact me on Discord velkog#2804, or on Twitter [@thevelkog](https://twitter.com/thevelkog).

Or if you have any improvements, feel free to make a pull request!  
