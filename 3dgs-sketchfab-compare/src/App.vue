<script setup lang="ts">
import SketchfabViewer from "./components/SketchfabViewer.vue";
import GaussianViewer from "./components/GaussianViewer.vue";
import GoogleForm from "./components/GoogleForm.vue";
import cameras from './assets/cameras.js';
import { ref } from "vue";

interface ViewerAPI {
  nextAnnotation: () => void;
  previousAnnotation: () => void;
  gotoAnnotation: (index: number) => void;
  getAnnotationList: (callback: (err: any, annotations: any[]) => void) => void;
}

const viewerRef = ref<ViewerAPI | null>(null);
const gaussianViewerRef = ref<{
  nextView: () => void;
  previousView: () => void;
} | null>(null);

const cameraState = ref("Camera")

const cameraViews = cameras;

const next = () => {
  gaussianViewerRef?.value?.nextView();
  viewerRef?.value?.previousAnnotation();
};
const previous = () => {
  gaussianViewerRef?.value?.previousView();
  viewerRef?.value?.nextAnnotation();
};

const dumpCamera = (data)=>{
  cameraState.value = (data);
}

defineExpose({
  cameraState
})
</script>

<template>

  <div class="app-container">
    <section class="viewer-section">
      <div class="columns">
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
          <div><code>position: {{ cameraState.position }} </code></div>
          <div><code>rotation: {{ cameraState.rotation }} </code></div>

          <div class="navigation-buttons">
            <button class="button" @click="next">Previous</button>
            <button class="button" @click="previous">Next</button>
          </div>
        </div>
        <div class="column">
          <GoogleForm />
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.viewer {
  overflow-clip-margin: content-box !important;
  overflow: clip !important;
  border-width: 2px;
  border-style: inset;
  border-color: initial;
  border-image: initial;
}

.viewer-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 0.5rem;
}

.navigation-buttons {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
  align-items: center;
  flex-direction: row;
  justify-content: center;
}

.nav-button {
  min-width: 300px;
  min-height: 50px;
  padding: 0.5rem 1rem;
  background-color: black;
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

.nav-button:disabled {
  background-color: #9e9e9e;
  cursor: not-allowed;
}
</style>
