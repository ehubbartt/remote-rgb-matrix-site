<script>
  import { Link } from "svelte-routing";
  import { onMount } from "svelte";
  import { initializeApp } from "firebase/app";
  import { getDatabase, ref, get, set } from "firebase/database";
  import { firebaseDb } from "../stores/firebase";
  let isOn = false;

  let db;

  onMount(() => {
    const unsubscribe = firebaseDb.subscribe((value) => {
      db = value;
    });

    const fetchIsOnState = async () => {
      const isOnRef = ref(db, "isOn");
      const snapshot = await get(isOnRef);
      isOn = snapshot.exists() ? snapshot.val() : false;
    };

    fetchIsOnState();

    return () => unsubscribe();
  });

  async function toggleOnOff() {
    const isOnRef = ref(db, "isOn");
    isOn = !isOn;
    await set(isOnRef, isOn);
  }
</script>

<main>
  <div class="button-container">
    <div class="button-primary" on:click={toggleOnOff}>
      {isOn ? "Turn Off" : "Turn On"}
    </div>
    <div class="button">
      <Link to="upload">Upload Image</Link>
    </div>
    <div class="button">
      <Link to="draw">Draw Image</Link>
    </div>
    <!-- <Link to="presets" class="button">Choose Preset</Link> -->
  </div>
</main>
