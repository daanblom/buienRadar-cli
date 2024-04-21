## ‼️ Since the recent update of buienradar.nl this script is no longer working, so i will archive this repository.
With the update the way the forecast is rendered completely changed. To to make the script work again i would basically need to start from scratch, which i might do another day!

## ☂️ buienRadar-cli 
### An unofficial command-line tool to display the rain forecast from the popular Dutch website buienradar.nl. The website typically displays images in a browser, while this script provides ASCII art in the terminal.

![Header image](img/buienradar-cli_header.gif?raw=true "Title")

_Please note that this script is an unofficial tool and its functionality is dependent on the website and its updates, thus the script may not always work as expected. This scipt has been tested on both Arch and Debian based linux distributions._

## 📓 Table Of Content :

- [Getting started](#-getting-started)
    - [Dependencies](#-dependencies)
    - [Downloading the script](#-downloading-the-script)
- [Usage](#-usage)
- [A brief explanation](#a-brief-explanation)
- [Known issues](#-known-issues)
- [License](#-license)
- [Links](#-links)

## ✨ Getting started:
Got your favourite Linux distro opened up? Command line open? Great! Now Please continue reading this README file👻.

## 🧱 Dependencies: 
The following dependencies are used and need to be installed for the script to work:
* [wget](http://www.gnu.org/software/wget/)
* [imagemagic](https://imagemagick.org/index.php)
* [ascii-image-converter (by TheZoraiz)](https://github.com/TheZoraiz/ascii-image-converter)

## 📥 Downloading the script: 
To begin, download the repository:
```bash
git clone https://github.com/daanblom/buienradar-cli
```
Navigate to the directory:
```bash
cd buienRadar-cli/
```
Make the script executable:
```bash
sudo chmod +x buienRadar-cli
```

## 🧪 Usage: 
Run the script by typing:
```bash
./buienRadar-cli
```
Alternatively, you can copy the script to the /usr/bin/ directory and run it from anywhere on the system, provided you have sudo permissions:
```bash
sudo cp buienRadar-cli /usr/bin/
```

## ☘️ A Brief explanation: 
While you are welcome to review the script, which is around a hundred lines, here is a brief breakdown of some of the code and thought processes:

- The first step is to fetch the image needed to create the ASCII animation from the [buienradar](https://www.buienradar.nl) website through a specifically generated URL. To do this, a couple of different calculations need to be done to generate timestamps that can be inserted in the URL to make sure we get a valid response.
- Research revealed that timestamps are:
	- Based on the last time the current time (in minutes only) could be divided by 20 (so in 5-minute blocks).
	- 15 minutes apart from each other.
	- In a different format and need to be reformatted from the standard Linux output, to "yearmonthdayhourmin" without any spaces or separators, and just using numbers.

This is done by the following piece of code:
```bash
ctime=$(date +%s)
round=$(echo "$ctime / 300 - 15" | bc)
timeFrom=$(echo "$round * 300" | bc)
timeTo=$(echo "$timeFrom + 900" | bc)
formatted_timeFrom=$(date -d @"$timeFrom" +"%Y%m%d%H%M")
formatted_timeTo=$(date -d @"$timeTo" +"%Y%m%d%H%M")
```

* These numbers then need to be inserted in the URL at the right place to get a valid response from the ```wget``` command and download the image we need.

The following code creates the variable that is the URL we need, with the timestamps injected at the right place:

```bash
url=$(echo "https://image-cdn.buienradar.nl/br-processing/image-api/RadarMapRainWebmercatorNL/Sprite/""$formatted_timeTo""__550x475_False_False_True_0_12_0_0_run""$formatted_timeFrom"".png")
```

- The downloaded image is a [sprite](https://en.wikipedia.org/wiki/Sprite_(computer_graphics)), we will eventually create a ```.gif``` that we can feed to ```ascii-image-converter```. To achieve this, the script first cuts up the sprite into separate frames. For this, the script uses a small ```while``` loop that cuts the image 12 evenly sized  ```.jpg```  ffiles and uses an alphabetical naming scheme to prevent sorting options that would place numbered files like ```somefile1.jpg``` and ```somefile10.jpg``` next to each other.
 
```bash
x=0
count=0
alphabet=({a..l})
while [ $count -lt $num_frames ]
do
    convert $image -crop ${square_width}x${size[1]}+$x+0 +repage $folder/${alphabet[$count]}.jpg
    x=$((x+square_width))
    count=$((count+1))
done
```

- After cutting the sprite into separate frames, the script then uses this single line of code to convert these frames into a .gif file. These files are stored in a hidden folder that is temporarily created by the script.

```bash
convert -delay $delay -loop 0 .bcli/*.jpg .bcli/out.gif
```

- As the final step, the script converts the image to an ASCII animation using a custom set of characters and displays it in the terminal.

```bash
charSet=".*:#oO\""
trap "rm -rf $folder" EXIT
ascii-image-converter -C -m "$charSet"  .bcli/out.gif
```

## 🚩 Known issues

The smoothness of the animation, such as the "flickering" that may occur between frames, is largely influenced by the type of terminal emulator used. In my experience, the animation has been smooth in [st](https://st.suckless.org/), good in [kitty](https://sw.kovidgoyal.net/kitty/), and average in [alacritty](https://github.com/alacritty/alacritty).


## 📜 Licence 

This code of this project is licensed under the terms of the MIT license.

## 🔗 Links 

- The original repository can be found at: [https://github.com/daanblom/buienradar-cli](https://github.com/daanblom/buienradar-cli)
- The website being used to fetch data: [https://www.buienradar.nl](https://www.buienradar.nl)

### See what keeps me busy at [daanblom.com](https://www.daanblom.com), or say hi and [send me an email](mailto:contact@daanblom.com)
