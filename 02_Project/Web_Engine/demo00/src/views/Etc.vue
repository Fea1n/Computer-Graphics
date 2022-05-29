<template>
  <div id="etc">
    <div id="full">
    <div id="container"></div>
    </div>
    <div class="iconBox" @click="enlarge"><img src="https://s4.ax1x.com/2022/01/27/7jEmTO.png" title="点击放大"></div>
  </div>
</template>


<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader'
import { BloomEffect, EffectComposer, ShaderPass, EffectPass, RenderPass } from "postprocessing"
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
import Stats from 'three/examples/jsm/libs/stats.module';


export default {
  data() {
    return {
      publicPath: process.env.BASE_URL,
      mesh: null,
      camera: null,
      scene: null,
      spotLight: null,
      renderer: null,
      controls: null,
      composer: null,
      stats: null,
      FullScreen: false, // 是否全屏
      // mixer:null
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    // 初始化
    init() {
      this.createScene() // 创建场景
      // this.loadGLTF() // 加载GLTF模型
      this.loadFBX() //加载FBX模型
      this.createLight() // 创建光源
      this.createCamera() // 创建相机
      this.createRender() // 创建渲染器
      this.createControls() // 创建控件对象
      this.addstats()
      this.createComposer() //创建后处理对象
      this.render() // 渲染
    },
    // 创建场景
    createScene() {
      this.scene = new THREE.Scene()
      // this.scene.background = new THREE.Color(0x000000)
      this.setEnvMap("001")
    },
    // 加载GLTF模型
    loadGLTF() {
      const THIS = this
      const loader = new GLTFLoader()
      // loader.load(`${THIS.publicPath}static/models/niguang/ninguang.gltf`, result=> {
      loader.load(`${THIS.publicPath}static/guitar/scene.gltf`, result => {
        const model = result.scene.children[0]
        model.traverse((n) => {
          if (n.isMesh) {
            n.castShadow = true
            n.receiveShadow = true
            //提高纹理的各向异性
            if (n.material.map)
              n.material.map.anisotropy = 100
          }
        }
        )
        this.scene.add(model)
        // this.render()
      })
    },
    //加载FBX模型
    loadFBX() {
      let loader = new FBXLoader()
      loader.load('/static/models/Genshin/lisa.fbx', result => {
        //         result.traverse((n) => {
        //   if (n.isMesh) {
        //     n.castShadow = true
        //     n.receiveShadow = true
        //     //提高纹理的各向异性
        //     if (n.material.map)
        //       n.material.map.anisotropy = 100
        //   }
        // }
        // )

        // this.mixer = new THREE.AnimationMixer( result );

				// let action = this.mixer.clipAction( result.animations[ 0 ] );
				// action.play();

				// result.traverse( function ( child ) {

				// 	if ( child.isMesh ) {

				// 		child.castShadow = true;
				// 		child.receiveShadow = true;

				// 	}

				// } );
        this.scene.add(result)
      })
       
    },
    // 创建光源
    createLight() {
      // 环境光
      // const ambientLight = new THREE.AmbientLight(0xffffff, 0.1) 
      // this.scene.add(ambientLight)
      // 半球光
      const hemisphereLight = new THREE.HemisphereLight(0xffeeb1, 0x080820, 2)
      this.scene.add(hemisphereLight)
      // 聚光灯(由于光线追踪需求，暂时设定为动态全局变量)
      this.spotLight = new THREE.SpotLight(0xffa95c, 4)
      this.spotLight.castShadow = true
      this.spotLight.shadow.bias = -0.0001
      this.spotLight.shadow.mapSize.width = 10000
      this.spotLight.shadow.mapSize.height = 10000
      this.scene.add(this.spotLight)
    },
    // 创建相机
    createCamera() {
      const element = document.getElementById('container')
      const width = element.clientWidth
      const height = element.clientHeight
      const k = width / height
      // PerspectiveCamera( fov, aspect, near, far )
      this.camera = new THREE.PerspectiveCamera(35, k, 0.1, 1000)
      this.camera.position.set(0, 3, 5) // 设置相机位置

      this.camera.lookAt(new THREE.Vector3(10, 40, 0))
      this.scene.add(this.camera)
    },
    // 创建渲染器
    createRender() {
      const element = document.getElementById('container')
      this.renderer = new THREE.WebGLRenderer({ antialias: true })

      // 设置渲染区域尺寸
      this.renderer.setSize(element.clientWidth, element.clientHeight)
      //阴影映射
      this.renderer.shadowMap.enabled = true
      // this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
      //色调映射
      this.renderer.toneMapping = THREE.ReinhardToneMapping
      this.renderer.toneMappingExposure = 3
      element.appendChild(this.renderer.domElement)
    },
    // 创建控件对象
    createControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    },
    //增加帧数显示
    addstats() {
      const element = document.getElementById('container')
      this.stats = new Stats();
      element.appendChild(this.stats.dom);
    },
    // 创建动画函数
    // createAnimate(){

    // },
    //创建后处理函数
    createComposer() {
      this.composer = new EffectComposer(this.renderer)
      this.composer.addPass(new RenderPass(this.scene, this.camera))

    },
    setEnvMap(hdr) {
      new RGBELoader().setPath("./files/hdr/").load(hdr + ".hdr", (texture) => {
        texture.mapping = THREE.EquirectangularReflectionMapping
        this.scene.background = texture
        this.scene.environment = texture
      });
    },
    // 渲染
    render() {
      requestAnimationFrame(this.render)
      this.spotLight.position.set(
        this.camera.position.x + 5,
        this.camera.position.y + 5,
        this.camera.position.z - 5
      )

      // let clock = new THREE.Clock();

      // let delta = clock.getDelta();

			// if ( this.mixer ) this.mixer.update( delta );
      this.composer.render(this.scene, this.camera)
      this.stats.update()
    },
    enlarge() {
      // let element = document.documentElement;//全屏放大
      let element = document.getElementById("container"); //需要全屏容器的id
      // 浏览器兼容
      if (this.FullScreen) {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.msExitFullscreen) {
          document.msExitFullscreen();
        }
      } else {
        if (element.requestFullscreen) {
          element.requestFullscreen();
        } else if (element.webkitRequestFullScreen) {
          element.webkitRequestFullScreen();
        } else if (element.mozRequestFullScreen) {
          element.mozRequestFullScreen();
        } else if (element.msRequestFullscreen) {
          element.msRequestFullscreen();
        }
      }
      this.FullScreen = !this.FullScreen;
      
    },
  }
}
</script>


<style>

#etc {
  position: relative;
}

#full {
  width: 100%;
  height:500px;
}
#container{
  width: 100%;
  height:100%;
}
#container canvas {
  width: 100%;
  height:100%;
}
.iconBox {
  cursor: pointer;
  position: absolute;
  top: 200px;
  right: 20px;
}
</style>