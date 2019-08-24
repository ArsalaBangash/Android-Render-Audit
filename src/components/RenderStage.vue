<template>
  <div class="d-flex my-2 justify-space-between">
    <div class="d-flex">
      <div :class="stage.replace('_', '-').toLowerCase() + '-stage'" class="stage-block mr-1" />
      <div class="body-2 mr-4">{{stageName}}: {{(roundedDuration)}}ms</div>
    </div>
    <v-icon class="float-right" :color="statusIcon.color">{{statusIcon.name}}</v-icon>
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
    status: Number
  },
  computed: {
    roundedDuration() {
      return this.round(this.duration / 1000, 2);
    },
    statusIcon() {
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
</style>