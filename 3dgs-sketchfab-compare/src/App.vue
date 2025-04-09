<script setup>
import SketchfabViewer from './components/SketchfabViewer.vue';
import GaussianViewer from './components/GaussianViewer.vue';
import { ref } from 'vue';

const viewerRef = ref(null);
const gaussianViewerRef = ref(null);

const cameraViews = [
  {
    name: 'Default View',
    position: { x: 10.719943604115596, y: -4.908698065119254, z: -19.469864307449498 },
    rotation: { x: -0.10391765437346483, y: -0.24742050609682814, z: -0.026700636810493374, w: 0.9629492667146181}
  },
  {
    name: 'Top View',
    position: {  x: 0.35193592419235237, y: 1.0055439035098597, z: -4.628968631530759} ,
    rotation: {x: 0.4191330573740113, y: 0.0059014432098459715, z: -0.002724403241718242, w: 0.9079015534799637}
  },
  {
    name: 'Lubang', 
    position: {x: 1.1622871244790622, y: 0.36230609770561495, z: -1.677699106052149},
    rotation: {x: -0.02669777365207973, y: -0.8476062129711616, z: -0.04284053797554354, w: 0.5282192962403724}
  }
]

  const next = () => {
    gaussianViewerRef?.value.nextView();
    viewerRef?.value.previousAnnotation()
  };
  const previous = () => {
    gaussianViewerRef?.value.previousView();
    viewerRef?.value.nextAnnotation()
  };

</script>

<template>
  <div class="app-container">
    <section class="viewer-section">
      <h1>Puri Klungkung</h1>
      <div class="viewer-container">
          <SketchfabViewer 
        ref="viewerRef"
        modelId="e4a96f7b1b1d42f3b940c88411c1c3f9"
        :show-list="false"
        :loop="true"
        width="800px"
        height="600px"
        title="My 3D Model"
      />
  <GaussianViewer class="viewer"
          ref="gaussianViewerRef"
          modelUrl="/gs/gs_50000.compressed.splat"
          width="800px"
          height="600px"
          :cameraViews="cameraViews"
          :controlButtons="false"
          :cameraControls="false"
          :loop="true"
        />
      
        
       
      
      </div>
      <div class="navigation-buttons">
        <button @click="next" class="nav-button">
          Previous
        </button>
        <button @click="previous" class="nav-button">
          Next
        </button>
      </div>
    </section>
  </div>
</template>

<style scoped>
.viewer{
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
