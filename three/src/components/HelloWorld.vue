<template>
  <div id="app">
    <div id="container"></div>
    <div id="text"></div>
  </div>
</template>

<script>
import * as THREE from 'three'
// import { OBJLoader, MTLLoader } from 'three-obj-mtl-loader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader'
// import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader'
import { DDSLoader } from 'three/examples/jsm/loaders/DDSLoader'
import { CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer.js'
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null,
      controls: null,
      text: null,
      classrooms: []
    }
  },
  methods: {
    init: function () {
      let container = document.getElementById('container')
      this.camera = new THREE.PerspectiveCamera(
        45,
        container.clientWidth / container.clientHeight,
        0.01,
        100000
      )
      this.camera.position.set(80, 80, 80)
      this.scene = new THREE.Scene()
      // 辅助坐标系
      var axisHelper = new THREE.AxisHelper(250)
      this.scene.add(axisHelper)

      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setSize(container.clientWidth, container.clientHeight)

      container.appendChild(this.renderer.domElement)
      window.addEventListener('resize', this.onWindowResize, false)
      // window.addEventListener('mousemove', this.handleMouseMove, false)
    },
    addControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    },
    render() {
      requestAnimationFrame(this.render)
      this.renderer.render(this.scene, this.camera)
    },
    async loadObj() {
      await this.getInfo()
      let _this = this
      var onProgress = function (xhr) {
        if (xhr.lengthComputable) {
          var percentComplete = (xhr.loaded / xhr.total) * 100
          console.log(Math.round(percentComplete, 2) + '% downloaded')
        }
      }

      var onError = function () {}

      var manager = new THREE.LoadingManager()
      manager.addHandler(/\.dds$/i, new DDSLoader())

      // comment in the following line and import TGALoader if your asset uses TGA textures
      // manager.addHandler( /\.tga$/i, new TGALoader() );

      // new MTLLoader(manager).setPath('/').load('2.mtl', function (materials) {
      //   materials.preload()

      //   new OBJLoader(manager)
      //     .setMaterials(materials)
      //     .setPath('/')
      //     .load(
      //       '2.obj',
      //       function (object) {
      //         console.log(object)
      //         object.position.y = 0
      //         _this.scene.add(object)
      //       },
      //       onProgress,
      //       onError
      //     )
      // })
      // const materials = new THREE.MeshLambertMaterial({
      //   color: 0x0000ff, //三角面颜色
      //   wireframe: true //网格模型以线条的模式渲染
      // })
      new OBJLoader(manager)
        // .setMaterials(materials)
        .setPath('/')
        .load(
          '2.obj',
          function (object) {
            object.children.forEach((item, index) => {
              if (_this.classrooms[index].status === 0) {
                item.material = new THREE.MeshLambertMaterial({
                  color: 0xd54062
                })
                console.log(item)
              }else  if (_this.classrooms[index].status === 1) {
                item.material = new THREE.MeshLambertMaterial({
                  color: 0x799351
                })
                console.log(item)
              }else  if (_this.classrooms[index].status === 2) {
                item.material = new THREE.MeshLambertMaterial({
                  color: 0xffa36c
                })
                console.log(item)
              }
            })
            object.position.y = 0
            _this.scene.add(object)
          },
          onProgress,
          onError
        )
    },
    addLight() {
      // 点光源
      var point = new THREE.PointLight(0xffffff, 2)
      point.position.set(100, 50, 100) //点光源位置
      this.scene.add(point) //点光源添加到场景中
      var pointLightHelper = new THREE.PointLightHelper(point, 1)
      this.scene.add(pointLightHelper)

      // 环境光
      // var ambient = new THREE.AmbientLight(0xffffff)
      // this.scene.add(ambient) //环境光对象添加到scene场景中
    },
    makeTextSprite(message, parameters) {
      if (parameters === undefined) parameters = {}
      var fontface = Object.prototype.hasOwnProperty.call(
        parameters,
        'fontface'
      )
        ? parameters['fontface']
        : 'Arial'
      // var fontface = parameters.hasOwnProperty('fontface')
      //   ? parameters['fontface']
      //   : 'Arial'

      /* 字体大小 */
      // var fontsize = parameters.hasOwnProperty('fontsize')
      //   ? parameters['fontsize']
      //   : 18
      var fontsize = Object.prototype.hasOwnProperty.call(
        parameters,
        'fontsize'
      )
        ? parameters['fontsize']
        : 18

      /* 边框厚度 */
      // var borderThickness = parameters.hasOwnProperty('borderThickness')
      //   ? parameters['borderThickness']
      //   : 4
      var borderThickness = Object.prototype.hasOwnProperty.call(
        parameters,
        'borderThickness'
      )
        ? parameters['borderThickness']
        : 4

      /* 边框颜色 */
      // var borderColor = parameters.hasOwnProperty('borderColor')
      //   ? parameters['borderColor']
      //   : { r: 0, g: 0, b: 0, a: 1.0 }
      var borderColor = Object.prototype.hasOwnProperty.call(
        parameters,
        'borderColor'
      )
        ? parameters['borderColor']
        : { r: 0, g: 0, b: 0, a: 1.0 }

      /* 背景颜色 */
      // var backgroundColor = parameters.hasOwnProperty('backgroundColor')
      //   ? parameters['backgroundColor']
      //   : { r: 255, g: 255, b: 255, a: 1.0 }
      var backgroundColor = Object.prototype.hasOwnProperty.call(
        parameters,
        'backgroundColor'
      )
        ? parameters['backgroundColor']
        : { r: 255, g: 255, b: 255, a: 1.0 }

      /* 创建画布 */
      var canvas = document.createElement('canvas')
      var context = canvas.getContext('2d')

      /* 字体加粗 */
      context.font = 'Bold ' + fontsize + 'px ' + fontface

      /* 获取文字的大小数据，高度取决于文字的大小 */
      var metrics = context.measureText(message)
      var textWidth = metrics.width

      /* 背景颜色 */
      context.fillStyle =
        'rgba(' +
        backgroundColor.r +
        ',' +
        backgroundColor.g +
        ',' +
        backgroundColor.b +
        ',' +
        backgroundColor.a +
        ')'

      /* 边框的颜色 */
      context.strokeStyle =
        'rgba(' +
        borderColor.r +
        ',' +
        borderColor.g +
        ',' +
        borderColor.b +
        ',' +
        borderColor.a +
        ')'
      context.lineWidth = borderThickness

      /* 绘制圆角矩形 */
      this.roundRect(
        context,
        borderThickness / 2,
        borderThickness / 2,
        2 * textWidth + borderThickness,
        2 * fontsize * 1.4 + borderThickness,
        6
      )

      /* 字体颜色 */
      context.fillStyle = '#FFF'
      context.fillText(message, borderThickness, fontsize + borderThickness)

      /* 画布内容用于纹理贴图 */
      var texture = new THREE.Texture(canvas)
      texture.needsUpdate = true

      var spriteMaterial = new THREE.SpriteMaterial({ map: texture })
      var sprite = new THREE.Sprite(spriteMaterial)

      console.log(sprite.spriteMaterial)

      /* 缩放比例 */
      sprite.scale.set(10, 5, 0)

      return sprite
    },
    add2dText() {
      var earthGeometry = new THREE.SphereBufferGeometry(1, 16, 16)
      var earthMaterial = new THREE.MeshPhongMaterial({
        specular: 0x333333,
        shininess: 5,
        normalScale: new THREE.Vector2(0.85, 0.85)
      })
      let earth = new THREE.Mesh(earthGeometry, earthMaterial)
      this.scene.add(earth)

      var earthDiv = document.createElement('div')
      earthDiv.className = 'label'
      earthDiv.textContent = 'Earth'
      earthDiv.style.marginTop = '-1em'
      var earthLabel = new CSS2DObject(earthDiv)
      earthLabel.position.set(0, 1, 0)
      earth.add(earthLabel)
    },
    addMessage() {
      this.text = this.makeTextSprite(`现在是${this.message}`, {
        fontsize: 20,
        borderColor: { r: 255, g: 0, b: 0, a: 0.4 } /* 边框黑色 */,
        backgroundColor: { r: 255, g: 255, b: 255, a: 0.2 } /* 背景颜色 */
      })
      this.text.center = new THREE.Vector2(0, 0)
      this.scene.add(this.text)
      this.text.position.set(10, 10, 0)
    },
    /* 绘制圆角矩形 */
    roundRect(ctx, x, y, w, h, r) {
      ctx.beginPath()
      ctx.moveTo(x + r, y)
      ctx.lineTo(x + w - r, y)
      ctx.quadraticCurveTo(x + w, y, x + w, y + r)
      ctx.lineTo(x + w, y + h - r)
      ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h)
      ctx.lineTo(x + r, y + h)
      ctx.quadraticCurveTo(x, y + h, x, y + h - r)
      ctx.lineTo(x, y + r)
      ctx.quadraticCurveTo(x, y, x + r, y)
      ctx.closePath()
      ctx.fill()
      ctx.stroke()
    },
    addCityText() {
      var loader = new THREE.FontLoader()
      let _this = this
      loader.load('Arvo_Regular.json', function (font) {
        var geometry = new THREE.TextGeometry('Hello three.js!', {
          font: font,
          size: 80,
          height: 5,
          curveSegments: 12,
          bevelEnabled: true,
          bevelThickness: 10,
          bevelSize: 8,
          bevelSegments: 5
        })
        var mat = new THREE.MeshPhongMaterial({
          color: 0xffe502,
          specular: 0x009900,
          shininess: 30,
          shading: THREE.FlatShading
        })
        var textObj = new THREE.Mesh(geometry, mat)
        _this.scene.add(textObj)
      })
    },
    async getInfo() {
      let _this = this
      await axios
        .get(
          ' http://49.234.121.178:7300/mock/5f36530edad41119bd229dda/example_copy/getInfo'
        )
        .then((res) => {
          if (res.status === 200) {
            _this.classrooms = res.data.data.classrooms
          }
        })
    },
    handleMouseMove(event) {
      // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
      var raycaster = new THREE.Raycaster()
      var mouse = new THREE.Vector2()
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
      // 通过摄像机和鼠标位置更新射线
      raycaster.setFromCamera(mouse, this.camera)
      // 计算物体和射线的焦点
      var intersects = raycaster.intersectObjects(
        this.scene.children[4].children
      )
      for (var i = 0; i < this.scene.children[4].children.length; i++) {
        this.scene.children[4].children[
          i
        ].material = new THREE.MeshLambertMaterial({
          color: 0xffffff
        })
        // intersects[i].object.material.color.set(0xffffff)
        // console.log(intersects[i].object.material)
        // intersects[i].object.material = new THREE.MeshBasicMaterial({
        //   color: 0x009e60
        // })
      }
      if (intersects.length) {
        // console.log(intersects[0].object.name)
        this.message = intersects[0].object.name
        var vertices = new Float32Array([
          ...intersects[0].object.geometry.attributes.position.array.slice(105)
        ])
        this.text.position.set(...vertices)

        intersects[0].object.material = new THREE.MeshLambertMaterial({
          color: 0x009e60
        })
      }

      this.renderer.render(this.scene, this.camera)
    },
    onWindowResize() {
      // let windowHalfX = window.innerWidth / 2
      // let windowHalfY = window.innerHeight / 2

      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()

      this.renderer.setSize(window.innerWidth, window.innerHeight)
    }
  },
  mounted() {
    this.init()
    this.render()
    this.addControls()
    this.loadObj()
    this.addLight()
    this.addMessage()

    // this.addCityText()
    // this.add2dText()
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#container {
  height: 100vh;
}
#text {
  color: red;
}
</style>
