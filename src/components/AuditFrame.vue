<template>
  <div class="audit-frame-container pt-2 pl-4">
    <div class="subtitle mb-8">Frame #{{index + 1}}</div>
    <div class="body-1">Start time: {{timestamp}}ms</div>
    <div class="body-1 mb-8">Duration: {{duration}}ms</div>

    <div class="body-2" v-show="errorStageCount > 0">This frame has warnings</div>

    <div class="d-flex mt-4 render-pipeline-container">
      <div class="d-flex flex-column pipeline-thread-container">
        <div class="subtitle">Main Thread</div>
        <v-btn small text class="inspect-button" color="primary">
          <v-icon left>search</v-icon>Inspect
        </v-btn>
        <div class="d-flex flex-column mr-4">
          <div
            v-for="(stageStats, stage) in mainThreadStagesStats"
            :key="stage"
            class="d-flex flex-column"
          >
            <render-stage
              :stage="stage"
              :stageName="getNameForStage(stage)"
              :duration="frame.renderStageDurations[stage]"
              :status="getStageStatus(stage)"
              :hasWarning="!durationWithinStandardDeviation(stage)"
              :hasError="!durationWithinThreshold(stage)"
              :isGood="getStageStatus(stage) == 0"
            />
          </div>
        </div>
      </div>

      <div class="d-flex flex-column pipeline-thread-container">
        <div class="subtitle">Render Thread</div>
        <v-btn small text class="inspect-button" color="success">
          <v-icon left>search</v-icon>Inspect
        </v-btn>
        <div class="d-flex flex-column">
          <div
            v-for="(stageStats, stage) in renderThreadStagesStats"
            :key="stage"
            class="d-flex flex-column"
          >
            <render-stage
              :stage="stage"
              :stageName="getNameForStage(stage)"
              :duration="frame.renderStageDurations[stage]"
              :status="getStageStatus(stage)"
              :hasWarning="!durationWithinStandardDeviation(stage)"
              :hasError="!durationWithinThreshold(stage)"
              :isGood="getStageStatus(stage) == 0"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import RenderStage from "./RenderStage";

export const StageStatus = {
  GOOD: 0,
  WARNING: 1,
  ERROR: 2
};

export const StageThresholds = {
  VSYNC_DELAY: 4000,
  INPUT: 8000,
  ANIMATION: 8000,
  MEASURE_LAYOUT: 8000,
  DRAW: 8000,
  SYNC_UPLOAD: 4000,
  COMMAND_ISSUE: 8000,
  SWAP_BUFFERS: 8000
};

export default {
  name: "audit-frame",
  components: {
    RenderStage
  },
  props: {
    frame: Object,
    index: Number,
    stats: Object,
    timestamp: Number
  },
  mounted() {},
  computed: {
    duration: function() {
      return ((this.frame.endUs - this.frame.startUs) / 1000).toFixed();
    },
    mainThreadStagesStats: function() {
      return {
        MISC_TIME: this.$props.stats["MISC_TIME"],
        INPUT: this.$props.stats["INPUT"],
        ANIMATION: this.$props.stats["ANIMATION"],
        MEASURE_LAYOUT: this.$props.stats["MEASURE_LAYOUT"],
        DRAW: this.$props.stats["DRAW"]
      };
    },
    renderThreadStagesStats: function() {
      return {
        SYNC_UPLOAD: this.$props.stats["SYNC_UPLOAD"],
        COMMAND_ISSUE: this.$props.stats["COMMAND_ISSUE"],
        SWAP_BUFFERS: this.$props.stats["SWAP_BUFFERS"]
      };
    },
    errorStageCount() {
      return Object.keys(this.stats).filter(
        stage => this.getStageStatus(stage) === StageStatus.ERROR
      ).length;
    }
  },
  watch: {
    timestamp: function(val, oldVal) {
      console.log("timestamp changed");
    }
  },
  methods: {
    getErrorStageCount() {
      return Object.keys(this.stats).filter(
        stage => getStageStatus(stage) === StageStatus.ERROR
      ).length;
    },
    getWarningStageCount() {
      return Object.keys(this.stats).filter(
        stage => getStageStatus(stage) === StageStatus.WARNING
      ).length;
    },
    getStageStatus(stage) {
      if (
        this.durationWithinThreshold(stage) &&
        this.durationWithinStandardDeviation(stage)
      ) {
        return StageStatus.GOOD;
      } else if (
        this.durationWithinThreshold(stage) &&
        !this.durationWithinStandardDeviation(stage)
      ) {
        return StageStatus.WARNING;
      }
      return StageStatus.ERROR;
    },
    durationWithinStandardDeviation(stage) {
      console.log('here')
      return (
        this.frame.renderStageDurations[stage] <
        (this.stats[stage].meanDurationUs + this.stats[stage].sdDurationUs)
      );
    },
    durationWithinThreshold(stage) {
      return this.frame.renderStageDurations[stage] < StageThresholds[stage];
    },
    getIconForStage(stage) {
      if (this.getStageStatus(stage) === StageStatus.GOOD) {
        return { name: "check_circle", color: "green" };
      } else if (this.getStageStatus(stage) === StageStatus.WARNING) {
        return { name: "warning", color: "orange" };
      } else if (this.getStageStatus(stage) === StageStatus.ERROR) {
        return { name: "error", color: "red" };
      }
    },
    getNameForStage(stage) {
      switch (stage) {
        case "MISC_TIME": {
          return "Misc Time"
        }
        case "INPUT": {
          return "Input";
        }
        case "ANIMATION": {
          return "Animation";
        }
        case "MEASURE_LAYOUT": {
          return "Measure & Layout";
        }
        case "DRAW": {
          return "Draw";
        }
        case "SYNC_UPLOAD": {
          return "Sync & Upload";
        }
        case "COMMAND_ISSUE": {
          return "Command Issue";
        }
        case "SWAP_BUFFERS": {
          return "Swap Buffers";
        }
      }
    }
  },
  data() {
    return {};
  }
};
</script>

<style >

.pipeline-thread-container {
  min-width: 236px;
}
.stage-bar {
  height: 12px;
  border-radius: 2px;
  max-width: 124px;
}

.mean-stage {
  background-color: #c0c2c9;
}

.audit-frame-container {
  background-color: white;
  height: 550px;
}

.stage-block {
  width: 18px;
  height: 18px;
  border-radius: 6px;
}

.v-btn {
  padding: 0px !important;
}
.v-btn__content {
  justify-content: start !important;
}

.inspect-button {
  padding: 6px !important;
  justify-content: start !important;
  margin: 0px !important;
  width: 100px;
}
.render-pipeline-container {
  width: 575px;
  justify-content: space-between;
}
.v-expansion-panel-header__icon {
  margin-left: 0px !important;
}

.large-panel {
  width: 550px !important;
}

.no-box-shadow {
  -webkit-box-shadow: none;
  -moz-box-shadow: none;
  box-shadow: none;
}
</style>