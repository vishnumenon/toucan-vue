<template>
  <div v-if="embeddedMode" style="width: 100%; height: 100%"></div>
</template>

<script>
function waitForToucan(resolve) {
  if (typeof ToucanAI !== "undefined") {
    resolve();
  }
  setTimeout(() => waitForToucan(resolve), 200);
}

function loadScript(src, alreadyLoaded) {
  return new Promise(function(resolve, reject) {
    if (alreadyLoaded) {
      waitForToucan(resolve);
    } else {
      var tag = document.createElement("script");
      tag.async = false;
      tag.src = src;
      tag.addEventListener("load", resolve);
      tag.addEventListener("error", reject);
      document.getElementsByTagName("body")[0].appendChild(tag);
    }
  });
}

export default {
  name: "ToucanAIChat",
  props: [
    "resourceRoot",
    "serverURL",
    "id",
    "secret",
    "tryToResume", // If true, try to resume an existing session on load, by checking cookies. Only takes effect if deployed is also true
    "deployed", // When deployed is false, no session is created. When deployed is true, either a new session is created or an existing one is resumed
    "loadConvo", // When non-null, load the provided convo ID and resume it. If deployed is true, this is ignored.
    "messageDelay", // in ms
    "onProductsSelected", // argument is an array of product objects
    "onProductClicked", // arguments are Product, Event. Return 'true' if the link should be followed after calling callback
    "onGoalReached", // arguments are the Product for which the goal was reached
    "onReady", // Fires when toucan is fully ready/rendered, with 'this' as argument
    "customStylesheet",
    "accentColor", // Defaults to "247, 71, 6", Must be a comma-separated RGB triplet string!!
    "embeddedMode", // When in embedded mode, only the message log renders; no FAB etc
    "allowInput", // When false, don't show input
    "hidePreviews",
    "windowTitle",
    "alreadyLoaded" // set to true on all but the first-loaded instance on a page
  ],
  mounted: function() {
    let widgetUrl =
      (this.resourceRoot || "https://dev.toucanai.com:91") + "/widget.js";
    loadScript(widgetUrl, this.alreadyLoaded).then(() => {
      let options = Object.assign({}, this.$props);
      // Delete any undefined keys
      Object.keys(options).forEach(
        key => options[key] === undefined && delete options[key]
      );
      if (this.embeddedMode) {
        options.parent = this.$el;
      }
      options.onReady = t => {
        this.toucanInstance = t;
        if (this.onReady) {
          this.onReady(t);
        }
      };
      new ToucanAI(options);
    });
  }
};
</script>

<style>
</style>
