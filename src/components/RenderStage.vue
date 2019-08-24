<template>
  <div class="d-flex my-2 justify-space-between">
    <div class="d-flex">
      <div :class="stage.replace('_', '-').toLowerCase() + '-stage'" class="stage-block mr-1" />
      <div class="body-2 mr-4">{{stageName}}: {{(roundedDuration)}}ms</div>
    </div>
    <div class="d-flex">
      <v-tooltip v-if="isGood" bottom>
        <template v-slot:activator="{ on }">
          <v-icon class="default-cursor" color="green" v-on="on">check_circle</v-icon>
        </template>
        <span>Nothing wrong here!</span>
      </v-tooltip>
      <v-tooltip bottom v-if="hasWarning && !isGood && duration > 0">
        <template  v-slot:activator="{ on }">
          <v-icon class="default-cursor" color="orange" v-on="on">warning</v-icon>
        </template>
        <span>Deviated from most frames</span>
      </v-tooltip>
      <v-tooltip v-if="hasError && !isGood && duration > 0" bottom>
        <template v-slot:activator="{ on }">
          <v-icon class="default-cursor" color="red" v-on="on">error</v-icon>
        </template>
        <span>Exceeded our threshold</span>
      </v-tooltip>
      <!-- <v-icon  :color="statusIcon.color">{{statusIcon.name}}</v-icon> -->
    </div>
  </div>
</template>

<script>
import { StageStatus } from "./AuditFrame.vue";
export default {
  name: "render-stage",
  props: {
    stage: String,
    stageName: String,
    duration: Number,
    status: Number,
    hasWarning: Boolean,
    hasError: Boolean,
    isGood: Boolean
  },
  computed: {
    roundedDuration() {
      return this.round(this.duration / 1000, 2);
    },
    statusIcons() {
      console.log(this.$props.status);
      if (this.$props.status === StageStatus.GOOD) {
        return { name: "check_circle", color: "green" };
      } else if (this.$props.status === StageStatus.WARNING) {
        return { name: "warning", color: "orange" };
      } else if (this.$props.status === StageStatus.ERROR) {
        return { name: "error", color: "red" };
      }
    }
  },
  methods: {
    round(value, precision) {
      var multiplier = Math.pow(10, precision || 0);
      return Math.round(value * multiplier) / multiplier;
    }
  }
};
</script>

<style scoped>
.stage-block {
  width: 18px;
  height: 18px;
  border-radius: 6px;
}

.v-btn {
  padding: 0px;
}
.v-btn__content {
  justify-content: start;
}

.default-cursor {
  cursor: default;
}
</style>