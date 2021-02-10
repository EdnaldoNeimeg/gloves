<template>
	<div class="home fill-height">
		<v-container fluid fill-height class="py-0">
			<v-row justify="center" class="fill-height">
				<v-col cols="12" md="2" class="" style="border-right: 1px solid #ddd;">
					<v-list dense v-for="(part, i) in parts" :key="i" class="py-0">
						<v-list-group
							:value="false"
						>
							<template v-slot:activator>
								<v-list-item-title>{{part.name}}</v-list-item-title>
							</template>

							<v-list-item link class="pl-10" @click="setActivePanel(part, 'color')">
								<v-list-item-content>
									<v-list-item-title>Color</v-list-item-title>
								</v-list-item-content>
							</v-list-item>
							<v-list-item link class="pl-10">
								<v-list-item-content>
									<v-list-item-title>Style</v-list-item-title>
								</v-list-item-content>
							</v-list-item>
						</v-list-group>
					</v-list>
				</v-col>
				<v-col cols="12" md="10" class="py-0" style="position: relative; overflow: hidden;">
					<canvas class="fill-height" id="renderCanvas"></canvas>
					<v-btn @click="scene.debugLayer.show();" style="position: absolute; top: 5px; right: 5px;" class="debug-btn" color="primary" small>Debug</v-btn>
					<div v-if="activePanel" class="panels">
						<div v-if="activePanel.type == 'color'" class="panel">
							<div class="pa-4 font-weight-bold">Colors</div>
							<div
								v-for="color in activePanel.part.colors"
								:key="color"
								@click="setColor(color)"
								class="pa-6 d-inline-block ma-4"
								:style="{backgroundColor: color}"></div>
						</div>
					</div>
				</v-col>
			</v-row>
		</v-container>
	</div>
</template>
<script>
import * as BABYLON from 'babylonjs';
import 'babylonjs-loaders';

export default {
	name: 'Home',
	components: {},
	data() {
		return {
			scene: null,
			shadowGenerator:null,
			leatherBumpTexture:null,
			parts:[{
				name:'Shell back',
				primitive: 'shell_back',
				colors:[
					'#1a1717',
					'#a38f50',
					'#161718',
					'#810000',
					'#1c1717',
					'#581d06',
					'#894300',
					'#6e967d',
				]
			},{
				name:'Web',
				primitive: 'web',
				colors:[
					'#1a1717',
					'#a38f50',
					'#161718',
					'#810000',
					'#1c1717',
					'#581d06',
					'#894300',
					'#6e967d',
				]
			}],
			activePanel: null,
		}
	},
	mounted() {
		this.createScene().then(() => {
			this.loadModel()
		})
	},
	methods: {
		setActivePanel(part, type){
			this.activePanel = {
				part: part,
				type
			}
		},
		setColor(color){

			var meshes = this.scene.meshes.filter( m => {
				return m.name.includes(this.activePanel.part.primitive)
			})
			

			meshes.forEach( m => {
				var material = m.material

				var decColorArray = this.hexToDec(color);

				material.albedoColor = new BABYLON.Color3(decColorArray[0], decColorArray[1], decColorArray[2]);
				
				// var lbt
				
				// if(this.leatherBumpTexture){
				// 	lbt = this.leatherBumpTexture
				// } else {
				// 	lbt = new BABYLON.Texture("maps/leather-bump2.png", this.scene);
				// 	this.leatherBumpTexture = lbt
				// }

				// material.bumpTexture = lbt
				// material.bumpTexture.uScale = 8.0;
				// material.bumpTexture.vScale = 8.0;
				// material.roughness = 0.6;

				// m.material = material
			})
		},
		hexToDec(hexString){
			var str = hexString.replace('#','')
			var arr
			if(str.length == 3) {
				arr = str.split('')
				arr[0] += ''+arr[0] 
				arr[1] += ''+arr[1] 
				arr[2] += ''+arr[2] 
			} else {
				arr = str.match(/.{1,2}/g)
			}
			console.log(arr)
			var r =  [
				parseInt(arr[0], 16)/255,
				parseInt(arr[1], 16)/255,
				parseInt(arr[2], 16)/255,
			]

			return r
		},
		createScene() {
			var th = this
			return new Promise((resolve) => {
				// Get the canvas DOM element
				var canvas = document.getElementById('renderCanvas');
				// Load the 3D engine
				var engine = new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true });
				// CreateScene function that creates and return the scene
				var createScene = function() {
					// Create a basic BJS Scene object
					var scene = new BABYLON.Scene(engine);
					// Create a FreeCamera, and set its position to {x: 0, y: 5, z: -10}
					// var camera = new BABYLON.FreeCamera('camera1', new BABYLON.Vector3(0, 5, -10), scene);

					var camera = new BABYLON.ArcRotateCamera("camera1", 2.2, 0.8, 1.5, new BABYLON.Vector3(0, -0.2, 0), scene);
					// Target the camera to scene origin
					// camera.setTarget(BABYLON.Vector3.Zero());
					// Attach the camera to the canvas
					camera.attachControl(canvas, false);

					// Create a basic light, aiming 0, 1, 0 - meaning, to the sky
					// var hlight = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);

					// hlight.intensity = 5

					var light = new BABYLON.DirectionalLight("DirectionalLight", new BABYLON.Vector3(0, -1, 0), scene);
					light.intensity = 5
					// console.log(light)
					
					th.shadowGenerator = new BABYLON.ShadowGenerator(1024, light);

					// // Create a built-in "sphere" shape; its constructor takes 6 params: name, segment, diameter, scene, updatable, sideOrientation
					// var sphere = BABYLON.Mesh.CreateSphere('sphere1', 16, 2, scene, false, BABYLON.Mesh.FRONTSIDE);
					// // Move the sphere upward 1/2 of its height
					// sphere.position.y = 1;
					// // Create a built-in "ground" shape; its constructor takes 6 params : name, width, height, subdivision, scene, updatable
					// // var ground = BABYLON.Mesh.CreateGround('ground1', 6, 6, 2, scene, false);
					// // Return the created scene

					


					return scene;
				}
				// call the createScene function
				var scene = createScene();
				scene.clearColor = new BABYLON.Color4(0, 0, 0, 0);
				// run the render loop
				engine.runRenderLoop(function() {
					scene.render();
				});
				// the canvas/window resize event handler
				window.addEventListener('resize', function() {
					engine.resize();
				});

				this.scene = scene

				resolve()
			})
		},
		loadModel() {
			return new Promise((resolve) => {
				// Append glTF model to scene.
				BABYLON.SceneLoader.Append("3d/", "glove.gltf", this.scene, function(scene) {
					console.log(scene)
					
					scene.meshes.forEach( m => {
						// this.shadowGenerator.getShadowMap().renderList.push(m);
						// m.receiveShadows = true;
						
						if(m.material){
							m.material.backFaceCulling = false
						} else {
							var mat = new BABYLON.StandardMaterial('Mat' + Math.rand(), this.scene);
							mat.backFaceCulling = false
							m.material = mat
						}
					})


					// Create a default arc rotate camera and light.
					scene.createDefaultCameraOrLight(true, true, true);

					// The default camera looks at the back of the asset.
					// Rotate the camera by 180 degrees to the front of the asset.
					// scene.activeCamera.alpha += Math.PI;
				});
				resolve()
			})
		}
	}
}
</script>
<style lang="sass">
#renderCanvas
	width: 100% 
	&:focus
		outline: none

	.scene-explorer-host,
	.inspector-host
		position: absolute !important
.panels
	position: fixed
	bottom: 0
	left: 0
	width: 100%
	background-color: #f9f9f9
	border-top: 1px solid #eee
</style>