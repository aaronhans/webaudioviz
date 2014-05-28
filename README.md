web audio, waveforms, webgl tutorial
===========

![alt tag](https://raw.github.com/aaronhans/webaudioviz/master/examples/images/pattern.png)

# Access files via ajax

In order to load the audio file into the webpage you need to run some kind of server locally to serve the static files or launch your browser with security disabled

### Launch browser with security disabled

Launch Chrome with security disabled on Mac:

```
open -a Google\ Chrome --args --disable-web-security
```

This will allow you to just open an html file locally with the file open command and make ajax requests successfully. To do this on other OS see [stack overflow disable web security](http://stackoverflow.com/questions/3102819/disable-same-origin-policy-in-chrome)

### Or run local server

Install http://nodejs.org

Checkout this repository, from the root directory run

```
npm install
```

Then launch the server with 

```
npm start
```

Then you can browse everything in the examples directory at http://localhost:1337/ like http://localhost:1337/images/pattern.png

# Load Audio

This is a vanilla ajax request with responseType is set to arraybuffer and when data is received the global AudioContext 

```
function loadSound(url, callback) {
  var request = new XMLHttpRequest();
  request.open('GET', url, true);
  request.responseType = 'arraybuffer';

  request.onload = function() {
    context.decodeAudioData(request.response, function(buffer) {
      callback(buffer);
    }, onError);
  }
  request.send();

  function onError() {
  	console.log('error')
  }
}
```

# Play Audio

Create a web audio analyser node and use this to process the audio signal which will give us access to frequency data described in the [MDN Analyser Node docs](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode)

```
function playSound(buffer) {
  source = context.createBufferSource(); // creates a sound source
  source.buffer = buffer;                // tell the source which sound to play
  source.connect(context.destination);   // connect the source to the context's destination (the speakers)

	analyser = context.createAnalyser();
	source.connect(analyser);
	analyser.connect(context.destination);

  source.start(0);
}
```

# Three.js setup

The following are the basic requirements for a three.js to begin drawing

```
    var scene, camera, renderer, line;
    var geometry, material, mesh;

    function init() {

        scene = new THREE.Scene();

        camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 1, 10000 );
        camera.position.z = 1000;

        renderer = new THREE.WebGLRenderer();
        renderer.setSize( window.innerWidth, window.innerHeight );

        document.body.appendChild( renderer.domElement );

    }
```


# Animation

Three.js includes the request animation frame shim so you can access the browsers native loop function tied to the screen refresh rate.

```
    function animate() {

      requestAnimationFrame( animate );
      newLine();

      renderer.render( scene, camera );

    }
```

# Waveform

Draw a line representing the audio volume over the full frequency range

```
    function newLine() {

    	scene.remove(line);

			var material = new THREE.LineBasicMaterial({
				color: 0x0000ff
			});

			var geometry = new THREE.Geometry();

			array = new Uint8Array(analyser.frequencyBinCount);
			analyser.getByteFrequencyData(array)

			var Ypoints = array;
			var xPoint = -2048;
			for(var i=0;i<Ypoints.length;i++) {
				geometry.vertices.push( new THREE.Vector3( xPoint, Ypoints[i], 0 ) );
				xPoint = xPoint + 4;
			}

			line = new THREE.Line( geometry, material );
			scene.add( line );
    }
```

Add calls to init and animate to your volume load function to begin drawing lines once you have audio data

```
  init();
  animate();
```

This should give you an animated view of the signal strength over the full spectrum

![alt tag](https://raw.github.com/aaronhans/webaudioviz/master/examples/images/freq.png)

We can hear between 20 and 20,000 Hz. The web audio api provides data over 44100 kHz sample size. You can define the fftSize which affects the number of slices of spectrum you receive in frequencyBinCount. I see a default fftSize of 1024 so it is slicing the spectrum into approximately 43Hz chunks. Here is a chart identifying the frequencies [corresponding to audible octaves](http://en.wikipedia.org/wiki/Audio_frequency)


