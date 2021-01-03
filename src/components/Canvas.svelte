<script>
  import { onMount } from "svelte";
  import fetch from 'node-fetch';
  import * as THREE from "three";
  import { STLLoader } from "three/examples/jsm/loaders/STLLoader.js";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

  let stlFile, fileinput;
  let stlSize = { x: 0, y: 0, z: 0 };
  let stlVolume = 0.0;

  // Necessary for camera/plane rotation
  let degree = Math.PI / 180;

  // Create scene
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x282c34);

  onMount(() => {
    const renderer = new THREE.WebGLRenderer({
      canvas: document.querySelector("canvas")
    });

    // There's no reason to set the aspect here because we're going
    // to set it every frame anyway so we'll set it to 2 since 2
    // is the the aspect for the canvas default size (300w/150h = 2)
    const camera = new THREE.PerspectiveCamera(70, 2, 1, 1000);
    camera.position.z = 100;
    camera.position.y = 50;
    camera.rotation.x = -45 * degree;

    const material = new THREE.MeshPhongMaterial({
      color: 0x555555,
      specular: 0xffffff,
      shininess: 50,
      shading: THREE.SmoothShading
    });

    // Add controls, targetting the same DOM element
    let controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);
    controls.rotateSpeed = 0.5;
    controls.update();

    // Plane
    // Ground (comment out line: "scene.add( plane );" if Ground is not needed...)
    var plane = new THREE.Mesh(
      new THREE.PlaneBufferGeometry(500, 500),
      new THREE.MeshPhongMaterial({
        color: 0x999999,
        specular: 0x101010
      })
    );
    plane.rotation.x = -90 * degree;
    plane.position.y = 0;
    plane.receiveShadow = true;
    scene.add(plane);

    // Lighting
    const frontSpot = new THREE.SpotLight(0xeeeece, 2, 0);
    const frontSpot2 = new THREE.SpotLight(0xddddce, 2, 0);
    frontSpot.position.set(1000, 1000, 1000);
    frontSpot2.position.set(-500, -500, 500);
    scene.add(frontSpot);
    scene.add(frontSpot2);

    function resizeCanvasToDisplaySize() {
      const canvas = renderer.domElement;
      const width = canvas.clientWidth;
      const height = canvas.clientHeight;
      if (canvas.width !== width || canvas.height !== height) {
        // you must pass false here or three.js sadly fights the browser
        renderer.setSize(width, height, false);
        camera.aspect = width / height;
        camera.updateProjectionMatrix();

        // set render target sizes here
      }
    }

    // Create an animate function, which will allow you to render your scene and define any movements
    const animate = function() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
      resizeCanvasToDisplaySize();
    };
    animate();
  });

  // https://stackoverflow.com/questions/13853301/why-doesnt-filereader-pass-file-to-loader-load-used-by-three-js-scene
  const onFileSelected = e => {
    let reader = new FileReader();
    let fileObject = e.target.files[0];

    reader.onload = function() {
      var loader = new STLLoader();
      //console.log(this.result);
      var geometry = loader.parse(this.result);
      //get stil volume
      getSize(geometry);
      getVolume(geometry);

      var material = new THREE.MeshPhongMaterial({
        ambient: 0xff5533,
        color: 0xff5533,
        specular: 0x111111,
        shininess: 200
      });

      var mesh = new THREE.Mesh(geometry, material);
      mesh.name = "loadedMeshObject";
      mesh.castShadow = true;
      mesh.receiveShadow = true;

      // model offset
      mesh.position.set(0, 0, 0);
      // modal orientation on load
      mesh.rotation.x = THREE.Math.degToRad(-90);
      mesh.rotation.z = THREE.Math.degToRad(-25);
      // Add loaded model to scene
      scene.add(mesh);
    };
    reader.readAsArrayBuffer(fileObject);
    //console.log("File Object:", fileObject);
  };

  function getSize(geometry) {
    let size = new THREE.Vector3();
    geometry.computeBoundingBox();
    geometry.boundingBox.getSize(size);
    //console.log("Size:", size)
    return (stlSize = size);
  }

  function getVolume(geometry) {
    let position = geometry.attributes.position;
    let faces = position.count / 3;
    let sum = 0;
    let p1 = new THREE.Vector3(),
      p2 = new THREE.Vector3(),
      p3 = new THREE.Vector3();
    for (let i = 0; i < faces; i++) {
      p1.fromBufferAttribute(position, i * 3 + 0);
      p2.fromBufferAttribute(position, i * 3 + 1);
      p3.fromBufferAttribute(position, i * 3 + 2);
      sum += signedVolumeOfTriangle(p1, p2, p3);
    }
    //console.log(sum)
    return (stlVolume = sum);
  }

  function signedVolumeOfTriangle(p1, p2, p3) {
    return p1.dot(p2.cross(p3)) / 6.0;
  }

  // Clear file input
  function clearFileInput(ctrl) {
    try {
      ctrl.value = null;
    } catch (ex) {}
    if (ctrl.value) {
      ctrl.parentNode.replaceChild(ctrl.cloneNode(true), ctrl);
    }
  }

  // Rest the Scene and clear file
  function resetScene() {
    const object = scene.getObjectByName("loadedMeshObject");
    stlSize = { x: 0, y: 0, z: 0 };
    stlVolume = 0.0;
    scene.remove(object);
    clearFileInput(fileinput);
  }

</script>

<style>
  .canvas-container {
    height: 500px;
  }
  canvas {
    width: 100%;
    height: 100%;
  }
</style>

<section class="mt-4 overflow-hidden text-gray-600 body-font">
  <div class="mx-auto">
    <div class="flex flex-wrap mx-auto">
      <div class="w-full mb-6 lg:w-1/2 lg:pr-10 lg:pb-6 lg:mb-0">
        <h2 class="text-sm tracking-widest text-gray-500 uppercase title-font">
          3DAddict
        </h2>
        <h1 class="mb-4 text-3xl font-medium text-gray-900 title-font">
          Calculate STL Price
        </h1>
        <div class="flex py-2 border-t border-gray-200">
          <span class="text-gray-500">Material</span>
          <span class="ml-auto text-gray-900">PLA</span>
        </div>
        <div class="flex justify-between py-2 border-t border-gray-200">
          <span class="text-gray-500">Color</span>
          <div class="">
            <button
              class="w-6 h-6 bg-yellow-600 border-2 border-gray-300 rounded-full focus:outline-none" />
            <button
              class="w-6 h-6 ml-1 bg-blue-700 border-2 border-gray-300 rounded-full focus:outline-none" />
            <button
              class="w-6 h-6 ml-1 bg-indigo-500 border-2 border-gray-300 rounded-full focus:outline-none" />
          </div>
        </div>
        <div class="flex py-2 mb-6 border-t border-b border-gray-200">
          <span class="text-gray-500">Layer Height</span>
          <span class="ml-auto text-gray-900">0.2</span>
        </div>

        <div
          class="flex flex-col items-end w-full mx-auto space-y-4 sm:flex-row">
          <div class="relative flex-grow w-full">
            <label for="full-name" class="text-sm leading-7 text-gray-600">
              Upload Stl File
            </label>
            <input
              type="file"
              on:change={e => onFileSelected(e)}
              bind:this={fileinput}
              class="w-full px-3 py-1 text-base leading-8 text-gray-700 transition-colors duration-200 ease-in-out bg-gray-100 bg-opacity-50 border border-gray-300 rounded outline-none focus:border-indigo-500 focus:bg-transparent focus:ring-2 focus:ring-indigo-200" />
          </div>
          <button
            class="w-full h-12 px-8 py-2 text-white uppercase bg-indigo-500 border-0 rounded focus:outline-none hover:bg-indigo-600 md:w-auto"
            on:click={resetScene}>
            reset
          </button>
        </div>
        <div class="flex flex-col items-start my-6">
          <div class="flex-grow">
            <h2 class="mb-1 text-lg font-medium text-gray-900 title-font">
              Stats
            </h2>
            <p class="text-base leading-relaxed">
              Size: {stlSize.x.toFixed(2)} x {stlSize.y.toFixed(2)} x {stlSize.z.toFixed(2)}
              mm
            </p>
            <p class="text-base leading-relaxed">
              Volume: {stlVolume.toFixed(2)} cm3
            </p>
          </div>
        </div>
        <div class="text-gray-600 body-font">
  <table class="w-full text-left whitespace-no-wrap table-auto">
    <thead>
      <tr>
        <th class="px-4 py-3 text-sm font-medium tracking-wider text-gray-900 bg-gray-100 rounded-tl rounded-bl title-font">Est. Cost</th>
        <th class="px-4 py-3 text-sm font-medium tracking-wider text-gray-900 bg-gray-100 title-font">Total Price</th>
        <th class="px-4 py-3 text-sm font-medium tracking-wider text-gray-900 bg-gray-100 title-font">Profit</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class="px-4 py-3">$0.40</td>
        <td class="px-4 py-3">$15.68</td>
        <td class="px-4 py-3">$15.28</td>
      </tr>
    </tbody>
  </table>
</div>
      </div>
      <div
        class="object-cover object-center w-full canvas-container lg:w-1/2 lg:h-auto">
        <canvas class="rounded" />
      </div>
    </div>
  </div>
</section>
