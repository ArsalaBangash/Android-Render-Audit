<template>
  <div>
    <div class="title mt-4 ml-4">Summary</div>
    <v-divider class="my-2" />

    <v-list class="mr-6">
      <v-list-group
        width="350px"
        v-for="(auditFrames, auditName) in renderAuditComponents"
        :key="auditName"
        v-model="isActive[auditName]"
        no-action
      >
        <template v-slot:activator>
          <v-list-item-content>
            <v-list-item-title v-text="names[auditName]"></v-list-item-title>
            <v-list-item-subtitle class="mt-2 mr-4">{{getSummary(auditName)}}</v-list-item-subtitle>
          </v-list-item-content>
          <v-tooltip width="350px" right>
            <template v-slot:activator="{ on }">
              <v-icon v-on="on">info</v-icon>
            </template>
            <span>{{auditDescriptions[auditName]}}</span>
          </v-tooltip>
        </template>

        <v-list-item
          width="350px"
          @click="setTimestamp(frame.startUs)"
          class="no-left-padding"
          v-for="frame in auditFrames"
          :key="frame.startUs"
        >
          <v-list-item-content>
            <div>{{namesSingular[auditName]}} at {{frame.startUs}}</div>
            <!-- <v-btn class="mt-2" flat small colo="primary">Inspect Range</v-btn> -->
          </v-list-item-content>
        </v-list-item>
      </v-list-group>
    </v-list>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";

export default {
  computed: {
    ...mapState({
      // arrow functions can make the code very succinct!
      renderAudit: state => state.renderAudit,
      renderAuditComponents: state => {
        return {
          longFrames: state.renderAudit.longFrames,
          tripleBuffers: state.renderAudit.tripleBuffers,
          skippedFrames: state.renderAudit.skippedFrames
        };
      }
    })
  },
  methods: {
    setTimestamp(timestamp) {
      this.$emit("set-timestamp", timestamp);
    },
    getSummary(auditName) {
      if (auditName == "tripleBuffers") {
        if (this.renderAuditComponents.tripleBuffers.length > 0) {
          return (
            "Your app triple buffered " +
            this.renderAuditComponents.tripleBuffers.length +
            " times."
          );
        }
        return "Your app did not triple buffer.";
      }
      if (auditName == "longFrames") {
        if (this.renderAudit.longFrames.length > 0) {
          return (
            this.renderAuditComponents.longFrames.length +
            " of your app's " +
            this.renderAudit.auditFrames.length +
            " frames were long."
          );
        }
        return "Your app had no long frames.";
      }
      if (auditName == "skippedFrames") {
        if (this.renderAudit.skippedFrames.length > 0) {
          return (
            "Your app skipped " +
            this.renderAuditComponents.skippedFrames.length +
            " frames."
          );
        }
        return "Your app had no skipped frames.";
      }
    }
  },
  data() {
    return {
      activeFrame: null,
      isActive: {
        tripleBuffers: false,
        longFrames: false,
        skippedFrames: false
      },
      names: {
        tripleBuffers: "Triple Buffers",
        longFrames: "Long Frames",
        skippedFrames: "Skipped Frames"
      },
      namesSingular: {
        tripleBuffers: "Triple Buffer",
        longFrames: "Long Frame",
        skippedFrames: "Skipped Frame"
      },
      auditDescriptions: {
        tripleBuffers:
          "When your app is pushing frames to the Android system faster than it can draw them to the surface, what your user sees will be one frame behind. Since user input is processed in the frames sent by your app to the Android system, lagging behind means your app will take longer to process user input.",
        longFrames:
          "When your app takes longer than 16ms to draw a frame, this means the 60fps mark we want to hit for smooth user performance is missed. We call these long frames, and inspecting these frames' rendering pipeline can help identify what caused it to take longer than 16ms.",
        skippedFrames:
          "We have an skipped frame when the Android system is ready to draw a frame to the surface, but your app hasn't given it a frame to draw. Analyzing the rendering pipeline for the frames that led up to this no-refresh can help identify what caused the frame to not be ready."
      }
    };
  }
};
</script>

<style scoped>
.no-left-padding {
  padding-left: 16px !important;
}
.tooltip-text {
  max-width: 450px;
}
</style>