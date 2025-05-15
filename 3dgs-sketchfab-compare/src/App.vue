<script setup lang="ts">
import SketchfabViewer from "./components/SketchfabViewer.vue";
import GaussianViewer from "./components/GaussianViewer.vue";
import GoogleForm from "./components/GoogleForm.vue";
import { ref, onMounted } from "vue";
import { type CameraView } from "./types/cameraview";
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';
import InfoModal from "./components/InfoModal.vue";

interface ViewerAPI {
  nextAnnotation: () => void;
  previousAnnotation: () => void;
  goToAnnotation: (index: number) => void;
  getAnnotationList: (callback: (err: any, annotations: any[]) => void) => void;
}

const questionnaireReady = ref(false);
const viewerRef = ref<ViewerAPI | null>(null);
const gaussianViewerRef = ref<{
  nextView: () => void;
  setCameraView: (cameraView: CameraView) => void;
  previousView: () => void;
} | null>(null);

const cameraState = ref({
  position: [0, 0, 0],
  rotation: [0, 0, 0, 1],
});

const cameraViews = ref<CameraView[]>([]);
const currentCameraView = ref<CameraView | null>(null);
const activeCameraName = ref<string | null>(null);


const notify = (text: string) => {
      toast(text, {
        autoClose: 5000,
        position: "bottom-center",
        theme: "dark",
      }); // ToastOptions
    }

onMounted(() => {
  import("./assets/cameras.json").then((module) => {
    const tmp = module.default; 
    const camerasData = tmp.map((camera: any) => {
      return {
        name: camera.name,
        position: camera.position,
        euler: camera.euler,
        description: camera.description,
        visited: false,
        isloading: false,
        progress: 0,
        isloaded: false
      };
    }); 
    cameraViews.value = camerasData;
    cameraViews.value[0].visited = true;
    cameraViews.value[1].visited = true;
    cameraViews.value[0].isloaded = true;
    cameraViews.value[1].isloaded = true;
    currentCameraView.value =  cameraViews.value[0];
    if (camerasData.length > 0) {
      activeCameraName.value = camerasData[0].name;
    }
    setCameraSketchfab(camerasData[0]);
  });
});

const setCameraSketchfab = (cameraView: CameraView) => {
  if (viewerRef.value) {
    const idx = cameraViews.value.findIndex(
      (view) => view.name === cameraView.name
    );
    if (idx === -1) {
      console.error("Camera view not found");
      return;
    }
    console.log("idx", idx);
    viewerRef.value.goToAnnotation(idx);
  }
};

const setCamera3DGS = (cameraView: CameraView) => {
  if (gaussianViewerRef.value) {
    gaussianViewerRef.value.setCameraView(cameraView);
  }
};

const enableNextCameraButton = async (idx: number): Promise<void> => {
  // Initialize loading state for the current camera
  cameraViews.value[idx].isloading = true;
  cameraViews.value[idx].progress = 0;
  
  // Calculate timing parameters
  const totalTime = 1000 * 30; // 3 minutes in milliseconds
  const updateInterval = 10; // Update every 1 second
  const totalSteps = totalTime / updateInterval;
  
  // Create a promise that resolves when the loading is complete
  return new Promise<void>((resolve) => {
    let currentStep = 0;
    
    const loadingInterval = setInterval(() => {
      currentStep++;
      
      // Update the progress
      cameraViews.value[idx].progress = Math.min((currentStep / totalSteps) * 100, 100);
      
      // If we've reached the total steps, clear the interval and enable the next button
      if (currentStep >= totalSteps) {
        clearInterval(loadingInterval);
        cameraViews.value[idx].isloading = false;
        
        // Enable the next camera
        if (cameraViews.value[idx + 1]) {
          cameraViews.value[idx + 1].visited = true;
        }
        
        // Resolve the promise
        resolve();
      }

    }, updateInterval);

  });
};
const setCamera = (cameraView: CameraView) => {
  
  currentCameraView.value = cameraView;
  activeCameraName.value = cameraView.name;
  setCamera3DGS(cameraView);
  setCameraSketchfab(cameraView);
  const idx = cameraViews.value.findIndex(
    (view) => view.name === cameraView.name
  );
  if(isAllCamerasLoaded()) return;

  enableNextCameraButton(idx).then(() => {
     cameraViews.value[idx].isloaded = true;

    if(cameraViews.value[idx + 1]) {
      notify("Kamera " + (idx + 2) + " " + cameraViews.value[idx + 1].name + " sudah bisa dilihat");
    }else{
      notify("Semua kamera sudah dilihat. Saatnya meingisi kuisioner");
      console.log(cameraViews.value);
    }
    checkQuestionnareReady();
  });
};

const isAllCamerasLoaded = () => {
  return cameraViews.value.every((cameraView) => cameraView.isloaded);
};

const checkQuestionnareReady = () => {
  const allVisited = isAllCamerasLoaded();
  questionnaireReady.value = allVisited;
};

const dumpCamera = (data: any) => {
  cameraState.value = data;
};

defineExpose({
  cameraState,
  setCamera,
});
</script>

<template>
<div class="mt-0 pl-4">
  <InfoModal />

  <div class="columns">
    <div class="column">
      <h1 class="title mt-3">Kamera</h1>
      <p>Pilih posisi kamera untuk melihat model dari berbagai sudut.</p>
      <div v-for="(cameraView,idx) in cameraViews">
        <button 
          class="button mt-3 is-fullwidth" 
          :class="{ 'is-primary': activeCameraName === cameraView.name }" 
          :disabled="!cameraView.visited"
          @click="setCamera(cameraView)"
        >
          Kamera {{ idx + 1 }} (<b>{{ cameraView.name }}</b>)
        </button>
        <progress v-if="cameraView.isloading" class="progress mt-1 is-small is-primary" :value="cameraView.progress" max="100">Menuju kamera selanjutnya</progress>

      </div>
      <div class="mt-3">
        <article class="message  is-success">
  <div class="message-body">
              <h2 class="title">{{  currentCameraView?.name }}</h2>
<p class="mb-3">
   {{  currentCameraView?.description }}
 </p>
 <b>Klik dan tahan 3D model kemudian gerakkan mouse untuk melihat bentuk 3D dengan lebih detail. </b>
  </div>
</article>
      </div>
    </div>
    <div class="column">
      <SketchfabViewer
        ref="viewerRef"
        modelId="e4a96f7b1b1d42f3b940c88411c1c3f9"
        :show-list="false"
        :loop="true"
        width="939px"
        height="469px"
        title="My 3D Model"
      />
      <GaussianViewer
        class="viewer"
        ref="gaussianViewerRef"
        modelUrl="/gs/gs_50000.compressed.splat"
        :width="939"
        :height="469"
        :cameraViews="cameraViews"
        :controlButtons="false"
        :cameraControls="false"
        :loop="true"
        :focalLength="25.3083684786384"
        @frame="dumpCamera"
      />
    </div>
    <div class="column">
      <h1 class="title mt-3">Kuisioner</h1>
      <div v-if="questionnaireReady">
        <p>Silakan isi kuisioner untuk memberikan umpan balik tentang model 3D ini.</p>
        <GoogleForm />
      </div>
      <div v-else>
        <p>Silakan lihat semua kamera untuk membuka kuisioner.</p>
      </div>
      <footer><small>Halaman website ini adalah instrument penelitian. Tidak untuk disebarluaskan.</small></footer>

    </div>
  </div>
</div>
</template>


