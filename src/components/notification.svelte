<script>
  export let message = "";
  export let show = false;

  let timeout;

  $: if (show) {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      show = false;
    }, 2000); // Notification stays for 2 seconds
  }
</script>

<div class="notification" class:visible={show}>
  {message}
</div>

<style>
  .notification {
    position: fixed;
    top: -50px;
    left: 50%;
    transform: translateX(-50%);
    background-color: #4caf50; /* Green background */
    color: white;
    padding: 10px 20px;
    font-size: 16px;
    border-radius: 4px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    opacity: 0;
    transition: all 0.3s ease;
    z-index: 1000;
  }

  .notification.visible {
    top: 20px; /* Slide down */
    opacity: 1; /* Fade in */
  }
</style>
