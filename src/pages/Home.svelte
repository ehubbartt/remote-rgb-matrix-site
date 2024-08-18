<script>
  import { Link } from "svelte-routing";
  import { onMount } from "svelte";
  import { getDatabase, ref, get, set, onValue } from "firebase/database";
  import { firebaseDb } from "../stores/firebase";
  let isOn = false;
  let brightness = 100; // Default brightness value
  let db;
  let brightnessTimeout; // For the debounce mechanism
  let imageData = []; // To store the current image data
  let previewCanvas;
  let previewCtx;

  onMount(() => {
    const unsubscribe = firebaseDb.subscribe((value) => {
      db = value;
    });

    const fetchIsOnState = async () => {
      const isOnRef = ref(db, "isOn");
      const snapshot = await get(isOnRef);
      isOn = snapshot.exists() ? snapshot.val() : false;
    };

    const fetchBrightness = async () => {
      const brightnessRef = ref(db, "brightness");
      const snapshot = await get(brightnessRef);
      brightness = snapshot.exists() ? snapshot.val() : 100;
    };

    fetchIsOnState();
    fetchBrightness();

    // Set up image preview if canvas is ready
    if (previewCanvas) {
      previewCtx = previewCanvas.getContext("2d");
      fetchImageData();
    }

    return () => unsubscribe();
  });

  function fetchImageData() {
    const imageRef = ref(db, "matrixData");
    onValue(imageRef, (snapshot) => {
      if (snapshot.exists()) {
        imageData = snapshot.val();
        drawPreview(imageData);
      }
    });
  }

  function drawPreview(data) {
    if (!previewCtx || !previewCanvas) return; // Ensure the canvas is ready

    // Clear the canvas before drawing
    previewCtx.clearRect(0, 0, previewCanvas.width, previewCanvas.height);

    // Draw the image data onto the preview canvas
    for (let i = 0; i < 64; i++) {
      for (let j = 0; j < 64; j++) {
        const index = i * 64 + j;
        const [r, g, b] = data[index];
        previewCtx.fillStyle = `rgb(${r}, ${g}, ${b})`;
        previewCtx.fillRect(j, i, 1, 1);
      }
    }
  }

  async function toggleOnOff() {
    const isOnRef = ref(db, "isOn");
    isOn = !isOn;
    await set(isOnRef, isOn);
  }

  function updateBrightness(event) {
    brightness = event.target.value;

    if (brightnessTimeout) {
      clearTimeout(brightnessTimeout);
    }

    brightnessTimeout = setTimeout(async () => {
      const brightnessRef = ref(db, "brightness");
      await set(brightnessRef, brightness);
    }, 1000); // 1-second debounce delay
  }
</script>

<main>
  <div class="button-container">
    <!-- Image Preview -->
    <div class="image-preview">
      <canvas
        bind:this={previewCanvas}
        width="64"
        height="64"
        style="width: 128px; height: 128px; border: 1px solid black;"
      ></canvas>
    </div>

    <!-- Brightness Slider -->
    <div class="brightness-control">
      <label for="brightness-slider">Brightness: {brightness}</label>
      <input
        id="brightness-slider"
        type="range"
        min="1"
        max="100"
        bind:value={brightness}
        on:input={updateBrightness}
      />
    </div>

    <!-- Toggle On/Off Button -->
    <div class="button-primary" on:click={toggleOnOff}>
      {isOn ? "Turn Off" : "Turn On"}
    </div>

    <!-- Navigation Buttons -->
    <div class="button">
      <Link to="upload">Upload Image</Link>
    </div>
    <div class="button">
      <Link to="draw">Draw Image</Link>
    </div>
    <!-- <Link to="presets" class="button">Choose Preset</Link> -->
  </div>
</main>

<style>
  .button-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
  }

  .image-preview {
    margin-bottom: 20px;
    border: 1px solid black;
    width: 128px;
    height: 128px;
  }

  .brightness-control {
    margin-bottom: 20px;
    text-align: center;
  }

  #brightness-slider {
    width: 100%;
    max-width: 300px;
  }
</style>
