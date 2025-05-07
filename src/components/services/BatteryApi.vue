<template>
  <Generic :item="item">
    <template #content>
      <p class="title is-4">{{ item.name }}</p>
      <p class="subtitle is-6">
        <template v-if="present">
          <template v-if="item.subtitle">
            {{ item.subtitle }}
          </template>
        </template>
        <template v-else>No Battery Present :D</template>
      </p>
    </template>
    <template #indicator>
      <template v-if="present">
        <div class="status" :class="statusClass" :title="statusText">
          {{ statusText }}
        </div>
      </template>
      <template v-else>
        <div class="status error" title="No Battery Present">:D</div>
      </template>
    </template>
  </Generic>
</template>

<script>
// Parts of the code are taken from Ping and OctoPrint service
import service from "@/mixins/service.js";

export default {
  name: "BatteryApi",
  mixins: [service],
  props: {
    item: Object,
  },
  data: () => ({
    present: null,
    technology: null,
    health: null,
    plugged: null,
    status: null,
    temperature: null,
    voltage: null,
    current: null,
    // currentAverage: null,
    percentage: null,
    // level: null,
    // scale: null,
    // chargeCounter: null,
    // energy: null,
  }),
  computed: {
    statusClass: function () {
      if (this.percentage < 40 && this.status != "CHARGING") {
        return "error";
      } else {
        switch (this.status) {
          case "FULL":
            return "ready";
          case "OFFLINE":
            return "offline";
          case "CHARGING":
            return "in-progress";
          case "DISCHARGING":
            return "out-progress";
          case "NOT_CHARGING":
            return "out-progress";
          case "UNKNOWN":
            return "error";
          default:
            return "pending";
        }
      }
    },
    statusText: function () {
      if (this.percentage < 40 && this.status != "CHARGING") {
        return "Low Battery";
      } else {
        switch (this.status) {
          case "FULL":
            return "Full Battery";
          case "OFFLINE":
            return "Offline";
          case "CHARGING":
            return "Charging";
          case "DISCHARGING":
            return "Discharging";
          case "NOT_CHARGING":
            return "Discharging";
          default:
            return "Unknown";
        }
      }
    },
  },
  created() {
    const updateInterval = parseInt(this.item.updateInterval, 10) || 0;
    if (updateInterval > 0) {
      setInterval(this.fetchBattery, updateInterval);
    }

    this.fetchBattery();
  },
  methods: {
    fetchBattery: async function () {
      try {
        const response = await this.fetch(`status`);
        this.present = response.present;
        this.technology = response.technology;
        this.health = response.health;
        this.plugged = response.plugged;
        this.status = response.status;
        this.temperature = response.temperature;
        this.voltage = response.voltage;
        this.current = response.current;
        this.currentAverage = response.current_average;
        this.percentage = response.percentage;
        this.level = response.level;
        this.scale = response.scale;
        this.chargeCounter = response.charge_counter;
        this.energy = response.energy;
      } catch (err) {
        this.staus = "OFFLINE";
        console.log(err);
      }
    },
  },
};
</script>

<style scoped lang="scss">
.out-progress:before {
  background-color: hsl(35, 70%, 70%);
  box-shadow: 0 0 5px 1px hsl(35, 70%, 70%);
  animation: red-pulse 1s alternate infinite;
}
@keyframes red-pulse {
  0% {
    background: rgba(255, 255, 255, 0.2);
    box-shadow:
      inset 0px 0px 10px 2px hsla(35, 100%, 50%, 0.3),
      0px 0px 5px 2px hsla(35, 100%, 50%, 0.2);
  }
  100% {
    background: rgba(255, 255, 255, 1);
    box-shadow:
      inset 0px 0px 10px 2px hsla(35, 100%, 50%, 0.5),
      0px 0px 15px 2px hsl(35, 100%, 50%);
  }
}
</style>
