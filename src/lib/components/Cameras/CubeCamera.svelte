<script lang="ts">
import * as THREE from "three";
import { get_scenes } from "$lib/utils/context";
import { onMount } from "svelte";
import type { Wrapping, TextureFilter, TextureDataType, TextureEncoding } from 'three/src/constants.js';
import type { DepthTexture } from 'three/src/textures/DepthTexture.js';

export let envMap : THREE.Texture | null = null;
export let size: number = 512;

export let wrapS: Wrapping | undefined = THREE.ClampToEdgeWrapping;
export let wrapT: Wrapping | undefined = THREE.ClampToEdgeWrapping;
export let magFilter: TextureFilter | undefined = THREE.LinearFilter;
export let minFilter: TextureFilter | undefined = THREE.LinearMipmapLinearFilter;
export let format: number | undefined = THREE.RGBAFormat; // RGBAFormat;
export let type: TextureDataType | undefined = THREE.UnsignedByteType; // UnsignedByteType;
export let anisotropy: number | undefined = 1; // 1;
export let depthBuffer: boolean | undefined = true; // true;
export let stencilBuffer: boolean | undefined = false; // false;
export let generateMipmaps: boolean | undefined = true; // true;
export let depthTexture: DepthTexture | undefined =undefined ;
export let encoding: TextureEncoding | undefined  =  THREE.sRGBEncoding;

export let near: number = 1;
export let far: number = 100000;

const cubeRenderTarget = new THREE.WebGLCubeRenderTarget( size, { 
wrapS:wrapS, wrapT:wrapT, magFilter: magFilter,  
format: format, type:type, anisotropy:anisotropy, depthBuffer:depthBuffer, 
stencilBuffer: stencilBuffer, generateMipmaps: generateMipmaps, 
minFilter: minFilter, depthTexture:depthTexture,encoding:encoding } );

const cubeCamera = new THREE.CubeCamera( near, far, cubeRenderTarget );
const { contextScenes } = get_scenes();

onMount(async () => {
    contextScenes.scene.add( cubeCamera );
    contextScenes.addBeforeRender (()=> {if (contextScenes.renderer) {cubeCamera.update (contextScenes.renderer, contextScenes.scene)}});
    envMap = cubeRenderTarget.texture ;
    })

$: if (cubeCamera) {
}
</script>

<slot envMap={envMap}/>