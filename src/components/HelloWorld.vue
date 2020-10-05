<template>
  <div id="container">x</div>
</template>

<script>
import * as THREE from "three";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { TransformControls } from "three/examples/jsm/controls/TransformControls.js";

export default {
  name: "HelloWorld",
  data() {
    return {
      cameraPersp: "",
      cameraOrtho: "",
      currentCamera: "",
      scene: "",
      renderer: "",
      mesh: "",
      control: "",
      orbit: "",
      stats: "",
    };
  },
  methods: {
    init: function() {
      var container = document.getElementById("container");

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      // this.renderer.setSize(container.clientWidth, container.clientHeight);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      container.appendChild(this.renderer.domElement);
      const aspect = container.innerWidth / container.innerHeight;

      this.cameraPersp = new THREE.PerspectiveCamera(50, aspect, 0.01, 30000);
      this.cameraOrtho = new THREE.OrthographicCamera(
        -600 * aspect,
        600 * aspect,
        600,
        -600,
        0.01,
        30000
      );
      this.currentCamera = this.cameraPersp;

      this.currentCamera.position.set(1000, 500, 1000);
      this.currentCamera.lookAt(0, 200, 0);

      this.scene = new THREE.Scene();
      this.scene.add(new THREE.GridHelper(1000, 10, 0x888888, 0x444444));

      var light = new THREE.DirectionalLight(0xffffff, 2);
      light.position.set(1, 1, 1);
      this.scene.add(light);

      var geometry = new THREE.BoxGeometry(200, 200, 200);
      var material = new THREE.MeshNormalMaterial();

      this.orbit = new OrbitControls(
        this.currentCamera,
        this.renderer.domElement
      );
      this.orbit.update();
      this.orbit.addEventListener("change", this.animate);

      this.control = new TransformControls(
        this.currentCamera,
        this.renderer.domElement
      );
      this.control.addEventListener("change", this.animate);

      this.control.addEventListener("dragging-changed", (event) => {
        this.orbit.enabled = !event.value;
      });

      var mesh = new THREE.Mesh(geometry, material);
      this.scene.add(mesh);
      this.onWindowResize();

      this.control.attach(mesh);
      this.scene.add(this.control);

      window.addEventListener("resize", this.onWindowResize, false);

      window.addEventListener("keydown", (event) => {
        switch (event.keyCode) {
          case 81: // Q
            this.control.setSpace(
              this.control.space === "local" ? "world" : "local"
            );
            break;

          case 16: // Shift
            this.control.setTranslationSnap(100);
            this.control.setRotationSnap(THREE.MathUtils.degToRad(15));
            this.control.setScaleSnap(0.25);
            break;

          case 87: // W
            this.control.setMode("translate");
            break;

          case 69: // E
            this.control.setMode("rotate");
            break;

          case 82: // R
            this.control.setMode("scale");
            break;

          case 67: // C
            var position = this.currentCamera.position.clone();

            this.currentCamera = this.currentCamera.isPerspectiveCamera
              ? this.cameraOrtho
              : this.cameraPersp;
            this.currentCamera.position.copy(position);

            this.orbit.object = this.currentCamera;
            this.control.camera = this.currentCamera;

            this.currentCamera.lookAt(
              this.orbit.target.x,
              this.orbit.target.y,
              this.orbit.target.z
            );
            this.onWindowResize();
            break;

          case 86: // V
            var randomFoV = Math.random() + 0.1;
            var randomZoom = Math.random() + 0.1;

            this.cameraPersp.fov = randomFoV * 160;
            this.cameraOrtho.bottom = -randomFoV * 500;
            this.cameraOrtho.top = randomFoV * 500;

            this.cameraPersp.zoom = randomZoom * 5;
            this.cameraOrtho.zoom = randomZoom * 5;
            this.onWindowResize();
            break;

          case 187:
          case 107: // +, =, num+
            this.control.setSize(this.control.size + 0.1);
            break;

          case 189:
          case 109: // -, _, num-
            this.control.setSize(Math.max(this.control.size - 0.1, 0.1));
            break;

          case 88: // X
            this.control.showX = !this.control.showX;
            break;

          case 89: // Y
            this.control.showY = !this.control.showY;
            break;

          case 90: // Z
            this.control.showZ = !this.control.showZ;
            break;

          case 32: // Spacebar
            this.control.enabled = !this.control.enabled;
            break;
        }
      });

      window.addEventListener("keyup", function(event) {
        switch (event.keyCode) {
          case 16: // Shift
            this.control.setTranslationSnap(null);
            this.control.setRotationSnap(null);
            this.control.setScaleSnap(null);
            break;
        }
      });
    },

    onWindowResize() {
      const aspect = window.innerWidth / window.innerHeight;

      this.cameraPersp.aspect = aspect;
      this.cameraPersp.updateProjectionMatrix();

      this.cameraOrtho.left = this.cameraOrtho.bottom * aspect;
      this.cameraOrtho.right = this.cameraOrtho.top * aspect;
      this.cameraOrtho.updateProjectionMatrix();

      this.renderer.setSize(window.innerWidth, window.innerHeight);

      this.animate();
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.currentCamera);
    },
  },
  mounted() {
    this.init();
    this.animate();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#container {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>
