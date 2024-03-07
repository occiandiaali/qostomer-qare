<script>
  import { onMount } from "svelte";

  import * as tf from "@tensorflow/tfjs";
  import * as tmImg from "@teachablemachine/image";

  /**
   * @type {HTMLVideoElement}
   */
  let videoElement;

  /**
   * @type {string}
   */
  let errorMessage;

  /**
   * @type {tmImg.CustomMobileNet}
   */
  let model;
  let loading = true;

  let percentage = "";
  let verdict = "";

  let backgroundColour = "#fff";
  let fontColour = "#000";

  const modelURL = "/src/model/model.json";
  const metadataURL = "/src/model/metadata.json";

  onMount(async () => {
    try {
      model = await tmImg.load(modelURL, metadataURL);
      const stream = await navigator.mediaDevices.getUserMedia({
        video: true,
      });
      videoElement.srcObject = stream;
      videoElement.play();
      setInterval(predict, 1000);
      loading = false;
    } catch (error) {
      console.error(error);
      errorMessage = "Camera access is required..";
      alert(`${errorMessage}`);
    }
  });

  $: if (verdict === "Good") {
    backgroundColour = "orange";
    fontColour = "#045187";
  } else if (verdict === "Bad") {
    backgroundColour = "red";
    fontColour = "#fff";
  } else {
    backgroundColour = "#fff";
    fontColour = "#000";
  }

  async function predict() {
    const predictions = await model.predict(videoElement);
    const [chosenPrediction] = predictions.sort(
      (a, b) => b.probability - a.probability
    );

    if (chosenPrediction) {
      percentage = (chosenPrediction.probability * 100).toFixed(2) + "%";
      verdict = classNameToLabel(chosenPrediction.className);
    }
  }

  /**
   * @param {string} className
   */
  function classNameToLabel(className) {
    switch (className) {
      case "Good":
        return "Good";
      case "Bad":
        return "Bad";
      default:
        return "Unsure";
    }
  }
</script>

<main style="background-color: {backgroundColour};color: {fontColour}">
  <h1>Monitor Page</h1>
  <video bind:this={videoElement} width="600" height="480" />
  {#if errorMessage}
    <p class="text-red-600">{errorMessage}</p>
  {:else if loading}
    <p>Loading...</p>
  {:else if percentage && verdict}
    <h3 class="text-center">{verdict} -> {percentage} certainty.</h3>
  {/if}
</main>

<style>
  h1,
  p {
    text-align: center;
  }
  main {
    width: 100%;
    height: 100vh;
    padding: 0;
    box-sizing: border-box;
    position: absolute;
  }

  video {
    display: block;
    margin: 20px auto;
    border: solid 1px lightcoral;
  }
</style>
