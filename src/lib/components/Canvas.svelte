<script lang="ts">
import * as THREE from 'three';
import { onDestroy, onMount } from 'svelte';
import { set_scenes, setRaycaster } from '$lib/utils/context.js';
import { ContextScenes } from '$lib/core/manager.js';
import {RaycasterManager} from '$lib/core/raycaster.js'
import { loadingManager } from '$lib/utils/loadingManager.js';

 

$: outerWidth = 0
$: innerWidth = 0
$: outerHeight = 0
$: innerHeight = 0

let el : HTMLElement;
let container :HTMLElement;
let camera : THREE.PerspectiveCamera ;
let raycasterManager : RaycasterManager;
let renderer : THREE.WebGLRenderer;

let INTERSECTED : THREE.Intersection;
let theta = 0;

const pointer = new THREE.Vector2();
const radius = 100;

const run = (fn) => fn();
const invalidate = () => {
  	if (contextScenes.frame ) return;
  	
    	frame = requestAnimationFrame(() => {
    		contextScenes.frame = null;
  			contextScenes.before_render.forEach(run);
  			if (  contextScenes.camera != null) {
        renderer.render(contextScenes.scene,  contextScenes.camera.target);
        //rotacionarTudo();
        render();
        }
  		});
    
	};

let SCENES = new ContextScenes (invalidate);
const contextScenes = set_scenes(SCENES);
let frame : number = 0;
let manager = loadingManager();

init();

// onMount/createScene
function init() {
    contextScenes.scene.background = new THREE.Color( 0xf0f0f0 );
    raycasterManager = setRaycaster(new RaycasterManager());  
}

//onMount / animate
function onWindowResize() {
  
    if (  contextScenes.camera != null) {
    contextScenes.camera.target.aspect = innerWidth / innerHeight;
    contextScenes.camera.target.updateProjectionMatrix();
    }
    renderer.setSize( innerWidth, innerHeight );
  }

//
function onPointerMove( event ) {
    pointer.x = ( event.clientX / innerWidth ) * 2 - 1;
    pointer.y = - ( event.clientY /innerHeight ) * 2 + 1;
}

function rotacionarTudo () {
  theta += 0.1;
    if (  contextScenes.camera != null) {
     
    contextScenes.camera.target.position.x = radius * Math.sin( THREE.MathUtils.degToRad( theta ) );
    contextScenes.camera.target.position.y = radius * Math.sin( THREE.MathUtils.degToRad( theta ) );
    contextScenes.camera.target.position.z = radius * Math.cos( THREE.MathUtils.degToRad( theta ) );
    contextScenes.camera.target.lookAt( contextScenes.scene.position );
    contextScenes.camera.target.updateMatrixWorld();
  }
}

//PASSA PARA before_render
function render() {
    if (  contextScenes.camera != null) {
    // find intersections
    raycasterManager.update (pointer, contextScenes.camera.target);
    }
}

//onMount
export const createScene = (el) => {
renderer = new THREE.WebGLRenderer({ antialias: true, canvas: el });
contextScenes.setRenderer (renderer);
raycasterManager.onCanvas (el , renderer );
}

//onMount 
const animate = () => {
  frame = requestAnimationFrame(animate);
  invalidate();
};

onMount(() => { 
  contextScenes.update (innerWidth , innerHeight);
  createScene(el)
  window.addEventListener( 'resize', onWindowResize );
  onWindowResize();
  animate();
  invalidate();  
});

$: if (renderer) {
 invalidate();
}
</script>

<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight  />

<div class="container" bind:this={container}>
<canvas bind:this={el} on:mousemove={onPointerMove}  on:resize={onWindowResize} class="background"/>
  {#if contextScenes.scene}
  <slot/>
  {/if}
</div>

<style>
	.background {
		content: '';
		position: absolute;
		background: white;
		width: 100%;
		height: 100%;
		top: 0;
	}
</style>