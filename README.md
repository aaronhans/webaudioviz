web audio, waveforms, webgl tutorial
===========

![alt tag](https://raw.github.com/aaronhans/webaudioviz/master/images/pattern.png)

# Run local server

In order to load the audio file into the webpage you need to run some kind of server locally to serve the static files or launch your browser with security disabled
# Load Audio

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
