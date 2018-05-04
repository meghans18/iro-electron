# Iowa Research Online - 10 Million Downloads
In 2018, Iowa Research Online celebrated 10 million downloads on their website. To commemorate this accomplishment, this application was made to showcase information about the database on an 80 inch touchscreen in the main library. The application includes a map of the world where users can click on each country and see how many downloads that country contributed to the total. It also contains a sidebar of information where users can learn information about Iowa Research Online and view "flipbook" samples of some of the most popular articles in [Iowa Research Online](http://ir.uiowa.edu/).
## Getting Started
To get a copy of the project up and running on your local machine, follow the instructions below.
### Installing
This project uses Electron, so you will first need to install Node.js and download this repository, then run the following commands.
```
# Navigate to the project folder
cd iro-electron
# Install dependencies
npm install
# Run the application
npm start
```
## Project Basics
This project switches between two different Electron windows to display either a map or a video featured in IRO. To show/hide windows:
```
#send a message to main.js from one of the html files
ipcRenderer.send('message')
#in main.js, include Electron and ipc
const electron = require('electron')
const ipcMain = require('electron').ipcMain
#open window
ipcMain.on('message', function() {
	win.show()
};
#hide window
ipcMain.on('message', function() {
	win.hide()
};
```
Messages sent by a window must match the message in main.js for each specific action.
To send a message from main.js to a browser window:
```
win.webContents.send('message');
```
Another feature used in this project is a timer for inactivity. This timer switches between the two windows when no user is interacting with it.
```
function idleTimer() {
#create variables for the timers
    var t, open, insideRotate;
#call resetTimer for anything that counts as activity from the user. for example:
	window.onload = resetTimer;
    window.onclick = resetTimer;
    countriesLayer.on('click', function() {
		resetTimer();
    });

#create a function that handles the rotation between windows and
#any other actions to take during inactivity
    function openChild() {
	#first do any actions that need to happen right after inactivity timer goes off
	#for example, hide html elements or pause a video (more actions in actual code)
		$('#htmlID').hide();
		video.pause();
	#this timer then sends the message to open the child window
        ipcRenderer.send('reset');
	#rotate is the main timer part for the openChild function
		var rotate = function(){
			//the looper function creates a popup loop on the map based on information in the 10m.js file
            var loop = countriesArray.length;
            var looper = function(){
				if (loop > 0) {
                    loop--;
                    country = countriesArray[loop][0];
                    downloads = countriesArray[loop][1];
                    popup.setLatLng(latLngData[country]);
                    popup.setContent('<div class="marker-title">' + country + '</div>' + downloads + ' downloads');
                    popup.openOn(map);
                    setTimeout(function() {
						map.closePopup()
					}, 5000);
                }
                else {
                    return rotate();
                }
                insideRotate = setTimeout(looper, 5000);
            };
            looper();
        };
        rotate();
	#the open timer opens the map every 7.5 minutes in this case
	#this is because the video will be playing for 5 minutes, then the map will display for 2.5 minutes.
	#the video automatically closes itself in the video.html file
        open = setInterval(function() {
            ipcRenderer.send('reset');
        }, 450000);
        map.setView([20, 11.5], 3);
    }

#the resetTimer function is called whenever there is activity. it clears all the other timers so they stop going
#the t timer runs the openChild (rotation) function whenever there hasn't been activity for 2.5 minutes
    function resetTimer() {
        clearTimeout(insideRotate);
        clearInterval(open);
        clearTimeout(t);
        t = setTimeout(openChild, 150000);  // time is in milliseconds
    }
};

idleTimer();
```
## Built With
* [Electron](https://electronjs.org/docs)