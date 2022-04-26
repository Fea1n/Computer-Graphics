<template>
  <div id="etc">
    <div id="container"></div>
  </div>
</template>


<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { BloomEffect, EffectComposer, EffectPass, RenderPass } from "postprocessing"

export default {
  data() {
    return {
      publicPath: process.env.BASE_URL,
      mesh: null,
      camera: null,
      scene: null,
      spotLight:null,
      renderer: null,
      controls: null,
      composer:null
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    // 初始化
    init() {
      this.createScene() // 创建场景
      this.loadGLTF() // 加载GLTF模型
      this.createLight() // 创建光源
      this.createCamera() // 创建相机
      this.createRender() // 创建渲染器
      this.createControls() // 创建控件对象
      this.createComposer() //创建后处理对象
      this.render() // 渲染
    },
    // 创建场景
    createScene() {
      this.scene = new THREE.Scene()
      this.scene.background = new THREE.Color(0x000000)
    },
    // 加载GLTF模型
    loadGLTF() {
      const THIS = this
      const loader = new GLTFLoader()
      loader.load(`${THIS.publicPath}static/models/little_chestnut/scene.gltf`, result=> {
        // model.scene.children[0].scale.set(20, 20, 20)
        const model = result.scene.children[0]
        model.traverse((n)=>
          {
            if(n.isMesh){
            n.castShadow = true;
            n.receiveShadow = true
            //提高纹理的各向异性
            if (n.material.map) n.material.map.anisotropy =100
            }
          }
        )
        this.scene.add(model)
        this.render()
        // console.log(model)
      })
    },
    // 创建光源
    createLight() {
      // 环境光
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.1) 
      this.scene.add(ambientLight)
      // 半球光
      const hemisphereLight = new THREE.HemisphereLight(0xffff55, 0x00ffff, 0.6)
      this.scene.add(hemisphereLight)
      // 聚光灯(由于光线追踪需求，暂时设定为动态全局变量)
      this.spotLight = new THREE.SpotLight(0xffa95c,4) 
      // spotLight.position.set(150, 150, 150)
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
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
      
      // 设置渲染区域尺寸
      this.renderer.setSize(element.clientWidth, element.clientHeight) 
      //阴影映射
      this.renderer.shadowMap.enabled = true 
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
      //色调映射
      this.renderer.toneMapping = THREE.ReinhardToneMapping
      this.renderer.toneMappingExposure = 3
      element.appendChild(this.renderer.domElement)
    },
    // 创建控件对象
    createControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    },
    // 创建动画函数
    // createAnimate(){

    // },
    //创建后处理函数
    createComposer(){
        this.composer = new EffectComposer(this.renderer)
        this.composer.addPass(new RenderPass(this.scene,this.camera))
        
        //辉光效果
        const effectPass = new EffectPass(this.camera,new BloomEffect({
          intensity:3,
          luminanceThreshold:0.8,
          width:100,
          height:100,
        }))

        // this.composer.addPass(effectPass)
    },
    // 渲染
    render() {
      requestAnimationFrame(this.render)
      this.spotLight.position.set(
        this.camera.position.x +5,
        this.camera.position.y +5,
        this.camera.position.z -5
      )
      this.composer.render(this.scene, this.camera)
      // this.render(this.scene,this.camera)
    },
  }
}
</script>


<style>
#container {
    height: 500px;
}
</style>