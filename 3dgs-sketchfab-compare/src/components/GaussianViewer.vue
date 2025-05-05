<template>
    <div class="gaussian-viewer" ref="containerRef">
      <template v-if="props.controlButtons">
          <div class="navigation-buttons" v-if="(cameraViews?.length ?? 0) > 0">
        <button @click="previousView()" class="nav-button">Previous</button>
        <button @click="nextView()" class="nav-button">Next</button>
      </div>
      </template>
      
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, onMounted, onUnmounted } from 'vue'
  import * as SPLAT from 'gsplat'
  
  interface CameraView {
    name: string
    position: { x: number; y: number; z: number }
    rotation: { x: number; y: number; z: number; w: number }
  }
  
  const props = defineProps<{
    modelUrl: string
    width?: string
    height?: string
    cameraViews?: CameraView[]
    loop?: boolean
    controlButtons?: boolean,
    cameraControls?: boolean
  }>()
  
  const containerRef = ref<HTMLDivElement | null>(null)
  const currentViewIndex = ref(0)
  let scene: SPLAT.Scene
  let camera: SPLAT.Camera
  let renderer: SPLAT.WebGLRenderer
  let controls: SPLAT.OrbitControls
  let animationFrameId: number
  
//   const containerStyle = computed(() => ({
//     width: props.width || '100%',
//     height: props.height || '400px'
//   }))
  
  const setCameraView = (view: CameraView) => {
    if (!camera) return
  
    // Update camera position using Vector3
    camera.position = new SPLAT.Vector3(view.position.x, view.position.y, view.position.z)
  
    // Set camera rotation directly using quaternion
    camera.rotation = new SPLAT.Quaternion(
      view.rotation.x,
      view.rotation.y,
      view.rotation.z,
      view.rotation.w
    )
  
    // Update controls if they exist
    // if (controls) {
    //   // Set a simple target in front of the camera
    //   // This is a simpler approach that doesn't rely on applyQuaternion
    //   const target = new SPLAT.Vector3(
    //     camera.position.x, 
    //     camera.position.y, 
    //     camera.position.z - 5 // Just look 5 units ahead on Z axis
    //   )
      
    //   //controls.target = target
    //   controls.update()
    // }
  }
  
  const nextView = () => {
    if (!props.cameraViews?.length) return
    
    if (currentViewIndex.value < props.cameraViews.length - 1) {
      currentViewIndex.value++
    } else if (props.loop) {
      currentViewIndex.value = 0
    }
    
    setCameraView(props.cameraViews[currentViewIndex.value])
  }
  
  const previousView = () => {
    if (!props.cameraViews?.length) return
    
    if (currentViewIndex.value > 0) {
      currentViewIndex.value--
    } else if (props.loop) {
      currentViewIndex.value = props.cameraViews.length - 1
    }
    
    setCameraView(props.cameraViews[currentViewIndex.value])
  }
  
  // Expose methods to parent component
  defineExpose({
    nextView,
    previousView,
    setCameraView
  })



  function setFOV(camera: SPLAT.Camera, fovDegrees: number): void {
    
    // Convert FOV in degrees to radians
    const fovRadians = fovDegrees * (Math.PI / 180);
    
    // Calculate focal length from FOV
    camera.data.setSize(1044, 1044); // Set the size of the camera
    camera.update(); // Update the projection matrix
    // For a perspective projection, focal length = (width/2) / tan(FOV/2)
    const fx = (camera.data.width / 2) / Math.tan(fovRadians / 2);
    
    // Set the focal lengths
    camera.data.fx = fx;
    camera.data.fy = fx;
    camera.update();
  }
  
  const initViewer = async () => {
    if (!containerRef.value) return
  
    scene = new SPLAT.Scene()
    camera = new SPLAT.Camera()
   // setFOV(camera, 10) // Set FOV to 45 degrees

    // Set initial position and rotation
    camera.position = new SPLAT.Vector3(0, 0, 5)
    camera.rotation = new SPLAT.Quaternion(0, 0, 0, 1) // Identity quaternion
    
    renderer = new SPLAT.WebGLRenderer()

    console.log('Camera:', camera)
    console.log('Renderer:', renderer)
    console.log('Scene:', scene)
    // Always create orbit controls for smooth transitions, 
    // but disable interaction if cameraControls is false
    controls = new SPLAT.OrbitControls(camera, renderer.canvas)
    
    // Disable user interaction if cameraControls is false
    // if (props.cameraControls === false) {
    //   controls.enabled = false
    // }
  
    // Set canvas size
    renderer.canvas.style.width = props.width || '100%'
    renderer.canvas.style.height = props.height || '400px'
  
    // Add canvas to container
    containerRef.value.appendChild(renderer.canvas)
  
    try {
      // Load the splat file
      const splat = await SPLAT.Loader.LoadAsync(props.modelUrl, scene, () => {})
      console.log('Splat loaded:', splat)
      //splat.scale = new SPLAT.Vector3(10, 10, 10) // Scale the model to 100x its original size
      //splat.applyScale() // Scale the model to 100x its original size
      // Set initial camera view if available
      if (props.cameraViews?.length) {
        setCameraView(props.cameraViews[0])
      }
  
      // Start animation loop
      const frame = () => {
        // Always update controls for smooth transitions
        controls.update()
        renderer.render(scene, camera)
        animationFrameId = requestAnimationFrame(frame)
        //console.log(camera.position, camera.rotation)
      }
  
      animationFrameId = requestAnimationFrame(frame)
    } catch (error) {
      console.error('Error loading Gaussian Splatting model:', error)
    }
  }
  
  onMounted(() => {
    initViewer()
  })
  
  onUnmounted(() => {
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId)
    }
    renderer?.dispose()
  })
  </script>
  
  <style scoped>

  
  .navigation-buttons {
    position: absolute;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 1rem;
  }
  
  .nav-button {
    padding: 0.5rem 1rem;
    background-color: #42b883;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.3s;
  }
  
  .nav-button:hover {
    background-color: #3aa876;
  }
  </style>