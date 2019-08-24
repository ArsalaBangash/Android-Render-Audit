<template>
  <v-app>
    <v-content>
      <div class="render-audit-container">
        <summary-panel @set-timestamp="setTimestamp" />
        <v-divider vertical></v-divider>

        <div ref="auditFramesContainer" class="audit-frames-container d-flex">
          <audit-frame
          class="mr-12"
            v-for="(frame, index) in auditFrames"
            :key="frame.startUs"
            :frame="frame"
            :index="index"
            :stats="stats"
            :timestamp="timestamp"
          />
        </div>
      </div>
      <v-divider />
    </v-content>
  </v-app>
</template>

<script>
import { mapState } from "vuex";
import SummaryPanel from "./components/SummaryPanel";
import AuditFrame from "./components/AuditFrame";

export default {
  name: "App",
  components: {
    SummaryPanel,
    AuditFrame
  },
  computed: {
    ...mapState({
      renderAudit: state => state.renderAudit,
      auditFrames: state => state.renderAudit.auditFrames,
      stats: state => state.renderAudit.renderStageStats
    })
  },
  methods: {
    setTimestamp: function(timestamp) {
      this.timestamp = timestamp;
      const a = [1, 2, 3, 4];
      const frameAfterTimestampIndex = this.auditFrames.findIndex(
        (val, index, obj) => {

          return val.startUs > this.timestamp;
        }
      );
      const container = this.$refs.auditFramesContainer;
      const scrollWidth = container.scrollWidth;
      const clientWidth = container.clientWidth;
      const scrollLeft = container.scrollLeft;

      const auditFrameScrollX = scrollWidth / this.auditFrames.length;
      container.scrollLeft = auditFrameScrollX * (frameAfterTimestampIndex - 1);
    }
  },
  data: () => ({
    timestamp: 0
  })
};
</script>

<style>
body {
  background-color: white;
}
:root {
  --misc-time-color: #08896d;
  --input-color: #0daa8e;
  --animation-color: #0eb799;
  --measure-layout-color: #34c3a9;
  --draw-color: #427af9;
  --sync-upload-color: #5accf5;
  --command-issue-color: #d91e14;
  --swap-buffers-color: #e47319;
}

.misc-time-stage {
  background-color: var(--misc-time-color);
}

.input-stage {
  background-color: var(--input-color);
}

.animation-stage {
  background-color: var(--animation-color);
}

.measure-layout-stage {
  background-color: var(--measure-layout-color);
}

.draw-stage {
  background-color: var(--draw-color);
}

.sync-upload-stage {
  background-color: var(--sync-upload-color);
}

.command-issue-stage {
  background-color: var(--command-issue-color);
}

.swap-buffers-stage {
  background-color: var(--swap-buffers-color);
}

.render-audit-container {
  display: flex;
  width: 100%;
  background-color: white;
}

.audit-frames-container {
  overflow-x: auto;
  height: 100%;
}
</style>