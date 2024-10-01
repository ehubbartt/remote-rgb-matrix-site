<script>
  import { Link } from "svelte-routing";
  import { onMount } from "svelte";
  import { ref, get, set, onValue } from "firebase/database";
  import { firebaseDb } from "../stores/firebase";
  import Notification from "../components/Notification.svelte";

  let isOn = false;
  let brightness = 100; // Default brightness value
  let db;
  let brightnessTimeout; // For the debounce mechanism
  let imageData = []; // To store the current image data
  let previewCanvas;
  let previewCtx;
  let lastUpdateMs = 0;
  let initialLoad = true; // To prevent triggering the notification on page load

  // For Notification
  let show = false;
  let notificationMessage = "";

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

    // Listen for changes in `lastUpdatedMs` to trigger notification
    const lastUpdatedMsRef = ref(db, "lastUpdateMs");
    onValue(lastUpdatedMsRef, (snapshot) => {
      if (snapshot.exists()) {
        const newValue = snapshot.val();

        // Only show the notification if it's not the initial load
        if (newValue !== lastUpdateMs && !initialLoad) {
          lastUpdateMs = newValue;
          showNotification("Matrix successfully updated!");
        }

        // After the first load, allow future notifications
        if (initialLoad) {
          initialLoad = false;
        }
      }
    });

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

  // Function to show the notification
  function showNotification(message) {
    notificationMessage = message;
    show = true;
    setTimeout(() => {
      show = false;
    }, 2000); // Hide notification after 2 seconds
  }
</script>

<main>
  <Notification {show} message={notificationMessage} />

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
