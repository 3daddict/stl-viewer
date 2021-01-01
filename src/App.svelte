<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js';
	import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

	// Necessary for camera/plane rotation
	let degree = Math.PI/180;

	// Create scene
	const scene = new THREE.Scene();
	scene.background = new THREE.Color(0x282c34);
	
	// Define a camera, set it to fill the browser window and position it
	// THREE.PerspectiveCamera(field of view, aspect ratio, near, far)
	//const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 10000);

	// https://stackoverflow.com/questions/29884485/threejs-canvas-size-based-on-container
	// There's no reason to set the aspect here because we're going
	// to set it every frame anyway so we'll set it to 2 since 2
	// is the the aspect for the canvas default size (300w/150h = 2)
	const camera = new THREE.PerspectiveCamera(70, 2, 1, 1000);
	camera.position.z = 100;
	camera.position.y = 50;
	camera.rotation.x = -45 * degree;

	function resizeCanvasToDisplaySize() {
	const canvas = renderer.domElement;
	// look up the size the canvas is being displayed
	const width = canvas.clientWidth;
	const height = canvas.clientHeight;

	// adjust displayBuffer size to match
		if (canvas.width !== width || canvas.height !== height) {
			// you must pass false here or three.js sadly fights the browser
			renderer.setSize(width, height, false);
			camera.aspect = width / height;
			camera.updateProjectionMatrix();

			// update any render target sizes here
		}
	}
	
	// Define a renderer, and set it to fill the browser window
	const renderer = new THREE.WebGLRenderer();
	renderer.setSize(window.innerWidth, window.innerHeight);

	// Add controls, targetting the same DOM element
	let controls = new OrbitControls(camera, renderer.domElement);
	controls.target.set(0, 0, 0);
	controls.rotateSpeed = 0.5;
	controls.update();

	// Plane
	// Ground (comment out line: "scene.add( plane );" if Ground is not needed...)
	var plane = new THREE.Mesh(
		new THREE.PlaneBufferGeometry(500, 500 ),
		new THREE.MeshPhongMaterial( { 
			color: 0x999999, 
			specular: 0x101010 
		})
	);
	plane.rotation.x = -90 * degree;
	plane.position.y = 0;
	plane.receiveShadow = true;
	scene.add( plane );

	// Lighting
	const frontSpot = new THREE.SpotLight(0xeeeece);
	const frontSpot2 = new THREE.SpotLight(0xddddce);
	frontSpot.position.set(1000, 1000, 1000);
	frontSpot2.position.set(-500, -500, 500);
	scene.add(frontSpot);
	scene.add(frontSpot2);

	// Create an animate function, which will allow you to render your scene and define any movements
	const animate = function () {
		requestAnimationFrame(animate);
		renderer.render(scene, camera);
		resizeCanvasToDisplaySize();
	};
	animate();

	// ASCII file - STL Import
	let loader = new STLLoader();
	loader.load('3DBenchy.stl', function (geometry) {
		// model material
		var material = new THREE.MeshLambertMaterial({ 
			color: 0xff3e00, 
		});
		var mesh = new THREE.Mesh(geometry, material);
		// model offset
		mesh.position.set( 0, 0, 0);
		// modal orientation on load
		mesh.rotation.x = THREE.Math.degToRad(-90);
		mesh.rotation.z = THREE.Math.degToRad(-25);
		// Add loaded model to scene
		scene.add(mesh);
	});

	onMount(() => {	
		// Get an element from the DOM and append renderer.domElement to it
		document.getElementById('threejs').appendChild(renderer.domElement);
	})
</script>

<main>
	<h1>STL!</h1>
	<section class="canvas-container">
		<div class="canvas" id="threejs" />
	</section>
</main>

<style>

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	.canvas-container {
		margin: 1rem 2rem;
	}

	.canvas {
		width: 100%;
	}
</style>