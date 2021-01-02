<script>
  import { onMount } from "svelte";
  import * as THREE from "three";
  import { STLLoader } from "three/examples/jsm/loaders/STLLoader.js";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

  let stlFile, fileinput;

  let stlSize = {
    x: 0,
    y: 0,
    z: 0
  };

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
    display: flex;
    height: 500px;
  }

  .canvas-container > * {
    flex: 1 1 50%;
  }

  #editor {
    font-family: monospace;
    padding: 0.5em;
    background: #444;
    color: white;
  }

  canvas {
    width: 100%;
    height: 100%;
  }
</style>

<section class="canvas-container">
  <div id="editor">
    <div class="file-container">
      <form>
        <input
          type="file"
          on:change={e => onFileSelected(e)}
          bind:this={fileinput} />
      </form>
      <div>
        <button on:click={resetScene}>Reset</button>
      </div>
      <div class="info-container">
        <p>
          Size: {stlSize.x.toFixed(2)} x {stlSize.y.toFixed(2)} x {stlSize.z.toFixed(2)}
          mm
        </p>
        <p>Volume: {stlVolume.toFixed(2)} mm3</p>
      </div>
      <div class="settings-container" />
    </div>
  </div>
  <div class="result">
    <canvas />
  </div>
</section>
