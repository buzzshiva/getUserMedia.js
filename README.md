#getUserMedia.js

getUserMedia.js is a cross-browser shim for the [getUserMedia() API](http://dev.w3.org/2011/webrtc/editor/getusermedia.html) (now a part of [WebRTC](http://www.webrtc.org/)) that supports accessing a local camera device from inside the browser. Where WebRTC support is detected, it will use the browser's native getUserMedia() implementation, otherwise a Flash fallback will be loaded instead.

As you can see in the [demo](http://addyosmani.github.com/getUserMedia.js/demo.html), what the shim provides is more than enough to create interactive applications that can relay device pixel information on to other HTML5 elements such as the canvas. By relaying, you can easily achieve tasks like capturing images which can be saved, applying filters to the data, or as shown in the demo, even perform tasks like facial detection.

The shim currently works in all modern browsers with support for IE still being evaluated (IE8+ support will at minimum be targeted). Note that the API for this project is still under development and is currently being tweaked. I may end up refactoring this into a jQuery plugin, but wish to keep it vanilla for the time-being.

##Walkthough

Getting the shim working is fairly straight-forward. First, include the ```getusermedia.js``` script in your page.

```javascript
<script src="js/getusermedia.js"></script>
```

Next, define some mark-up that we can use as a container for the video stream. Below you'll notice that a simple ```div``` has been opted for (as per our demo(. What will happen when we initialize the shim with it is we will either inject a ```video``` tag for use (if WebRTC is enabled) or alternatively an ```object``` tag if the Flash fallback needs to be loaded instead. Whilst most modern browsers will support the ```video``` tag, there is no reason to be using it here if your only interest is relaying the video data for further processing or use elsewhere.

```
<div id="webcam"></div>
```

Calling the shim is as simple as:

```
getUserMedia(options, success, error);
```

where ```options``` is an object containing configuration data, ```success``` is a callback executed when the stream is successfully streaming and ```error``` is a callback for catching stream or device errors.

We use the configuration object (```options``` in the above) to specify details such as the element to be used as a container,  (e.g ```webcam```), the quality of the fallback image stream (```85```) and a number of additional callbacks that can be further used to trigger behaviour.

For the most part, any callback beginning with ```on``` in the below example is a Flash-fallback specific callback. If you don't need to use it, feel free to exclude it from your code. 

```javascript
var options = {

			"audio": true,
			"video": true,

			// the element (by id) you wish to apply
			el: "webcam",

			extern: null,
			append: true,

			// height and width of the output stream
			// container

			width: 320,
			height: 240,

			mode: "callback",

			// the flash fallback Url
			swffile: "fallback/jscam_canvas_only.swf",

			// quality of the fallback stream
			quality: 85,
			context: "",

			debug: function () {},

			// callback for capturing the fallback stream
			onCapture: function () {
				window.webcam.save();
			},
			onTick: function () {},

			// callback for saving the stream, useful for
			// relaying data further.
			onSave: function (data) {},
			onLoad: function () {}
		};
```

Below is a sample ```success``` callback taken from the demo application, where we update the video tag we've injected with the stream data. Note that it's also possible to capture stream errors by executing calls from within ```video.onerror()``` in the example.

```javascript
		success: function (stream) {
			if (App.options.context === 'webrtc') {

				var video = App.options.videoEl;
				var vendorURL = window.URL || window.webkitURL;
				video.src = vendorURL ? vendorURL.createObjectURL(stream) : stream;

				video.onerror = function () {
					stream.stop();
					streamError();
				};

			} else {
				//flash context
			}
		}
```

At present the ```error``` callback for ```getUserMedia()``` is fairly simple and should be used to inform the user that either WebRTC or Flash were not present or an error was experienced detecting a local device for use.

##Credits
* getUserMedia() shim, demos: Addy Osmani
* Flash webcam access implementation: Robert Eisele
* Glasses positoning and filters for demo: Wes Bos

##Browsers

```getUserMedia()``` is natively supported in [Chrome Canary](http://tools.google.com/dlpage/chromesxs)(simply enable experimental MediaStream compatibility in chrome://flags/) and Opera.next ([Camera build](http://snapshot.opera.com/labs/camera/)). When using the shim, if support isn't detected you will be provided a Flash fallback. 

##Documentation

A stable version of the shim was only very recently completed but documentation will be added to the repo (or the wiki) shortly. 

##Spec references

* http://dev.w3.org/2011/webrtc/editor/getusermedia.html
* http://dev.w3.org/2011/webrtc/editor/webrtc.html (broader purpose)



