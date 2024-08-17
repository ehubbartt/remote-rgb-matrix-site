<script>
  import { Router, Route, Link } from "svelte-routing";
  import Home from "./pages/Home.svelte";
  import Upload from "./pages/Upload.svelte";
  import Draw from "./pages/Draw.svelte";
  import Presets from "./pages/Presets.svelte";

  let url = "";
</script>

<Router {url}>
  <!-- Home Button -->
  <div class="home-button">
    <Link to="/"><span class="home-text">Home</span></Link>
  </div>

  <div>
    <Route path="/" component={Home} />
    <Route path="/upload" component={Upload} />
    <Route path="/draw" component={Draw} />
    <!-- <Route path="/presets" component={Pressets} /> -->
  </div>
</Router>

<style>
  .home-button {
    position: fixed;
    width: 50px;
    top: 10px;
    left: 10px;
    padding: 10px 20px;
    font-size: 16px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    z-index: 1000; /* Ensure it stays on top */
  }

  .home-button:hover {
    background-color: #0056b3;
  }

  .home-text {
    color: white;
    text-decoration: none;
  }

  nav {
    display: none; /* Hides the nav since we're only using the Home button */
  }

  div {
    padding-top: 60px; /* Add padding to prevent content from being hidden under the Home button */
  }
</style>
