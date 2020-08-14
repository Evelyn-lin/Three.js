<template>
  <div class="hello">
    <div id="container"></div>
  </div>
</template>

<script>
import * as Three from 'three'
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { OBJLoader, MTLLoader } from 'three-obj-mtl-loader'
const OrbitControls = require('three-orbit-controls')(Three)
export default {
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null
    }
  },
  methods: {
    init: function() {
      let container = document.getElementById('container')
      this.camera = new Three.PerspectiveCamera(
        70,
        container.clientWidth / container.clientHeight,
        0.01,
        10
      )
      this.camera.position.z = 1

      this.scene = new Three.Scene()

      let geometry = new Three.BoxGeometry(0.2, 0.2, 0.2)
      let material = new Three.MeshNormalMaterial()

      this.mesh = new Three.Mesh(geometry, material)
      this.scene.add(this.mesh)

      this.renderer = new Three.WebGLRenderer({ antialias: true })
      this.renderer.setSize(container.clientWidth, container.clientHeight)
      container.appendChild(this.renderer.domElement)
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    },
    render: function() {
      requestAnimationFrame(this.render)
      this.mesh.rotation.x += 0.01
      this.mesh.rotation.y += 0.02
      this.renderer.render(this.scene, this.camera)
    },
    loadObj() {
      //包含材质
      new MTLLoader().setPath('../assets/models/').load('1.mtl', materials => {
        console.log('materials', materials)
        materials.preload()
        new OBJLoader()
          .setMaterials(materials)
          .setPath('../assets/models/')
          .load('1.obj', obj => {
            obj.scale.set(30, 30, 30)
            obj.position.set(0, 0, 0)
            this.scene.add(obj)
          })
      })
    }
  },
  mounted() {
    this.init()
    this.render()
    this.loadObj()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#container {
  height: 100vh;
}
</style>
