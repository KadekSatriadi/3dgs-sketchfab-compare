<script setup lang="ts">
import SketchfabViewer from "./components/SketchfabViewer.vue";
import GaussianViewer from "./components/GaussianViewer.vue";
import GoogleForm from "./components/GoogleForm.vue";
import { ref, onMounted } from "vue";
import { type CameraView } from "./types/cameraview";

interface ViewerAPI {
  nextAnnotation: () => void;
  previousAnnotation: () => void;
  goToAnnotation: (index: number) => void;
  getAnnotationList: (callback: (err: any, annotations: any[]) => void) => void;
}

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
const activeCameraName = ref<string | null>(null);

onMounted(() => {
  import("./assets/cameras.json").then((module) => {
    const camerasData = module.default;
    cameraViews.value = camerasData;
    if (camerasData.length > 0) {
      activeCameraName.value = camerasData[0].name;
    }
  });
});

const setCamera = (cameraView: CameraView) => {
    activeCameraName.value = cameraView.name;

  if (gaussianViewerRef.value) {
    gaussianViewerRef.value.setCameraView(cameraView);
  }
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
  <div class="columns">
    <div class="column">
      <h1 class="title mt-3">Kamera</h1>
      <p>Pilih posisi kamera untuk melihat model dari berbagai sudut.</p>
      <div v-for="(cameraView,idx) in cameraViews">
        <button 
          class="button mt-3 is-fullwidth" 
          :class="{ 'is-primary': activeCameraName === cameraView.name }" 
          @click="setCamera(cameraView)"
        >
          Kamera {{ idx + 1 }} (<b>{{ cameraView.name }}</b>)
        </button>
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
      <p>Silakan isi kuisioner untuk memberikan umpan balik tentang model 3D ini.</p>
      <GoogleForm />
      <footer><small>Halaman website ini adalah instrument penelitian. Tidak untuk disebarluaskan.</small></footer>

    </div>
  </div>
</div>
</template>


