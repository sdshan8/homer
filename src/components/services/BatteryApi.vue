<template>
  <Generic :item="item">
    <template #content>
      <p class="title is-4">{{ item.name }}</p>
      <p class="subtitle is-7">
        <template v-if="present">
          <template v-if="item.subtitle">
            {{ item.subtitle }}
          </template>
          <template v-else>
            <!-- Discharging -->
            <template v-if="statusText == 'Using'">
              <i
                class="fa-solid mr-1"
                :class="faDischargingIcon(percentage)"
              ></i>
              <span v-if="percentage">{{ percentage.toFixed() }}%</span>
              <span class="separator mx-1"> | </span>
              <i class="fa-solid mr-1 fa-bolt"></i>
              <span v-if="current">{{ -current.toFixed() / 1000 }}mA</span>
            </template>
            <!-- Charging -->
            <template v-if="statusText == 'Charging'">
              <i class="fa-solid mr-1" :class="faChargingIcon(plugged)"></i>
              <span v-if="percentage">{{ percentage.toFixed() }}%</span>
              <span class="separator mx-1"> | </span>
              <i class="fa-solid mr-1 fa-bolt"></i>
              <span v-if="current">{{ -current.toFixed() / 1000 }}mA</span>
            </template>
            <!-- Full -->
            <template v-if="statusText == 'Full'">
              <i class="fa-solid mr-1 fa-car-battery"></i>
              <span v-if="voltage">{{ voltage / 1000 }}V</span>
              <span class="separator mx-1"> | </span>
              <span v-if="temperature">
                <i class="fa-solid fa-temperature-low mr-1"></i>
                {{ temperature }}°C
              </span>
            </template>
          </template>
        </template>
        <!-- No Battery Present :D -->
        <template v-else>No Battery Present :D</template>
      </p>
    </template>
    <template #indicator>
      <template v-if="present">
        <div class="status" :class="statusClass" :title="statusText">
          {{ statusText.toLowerCase() }}
        </div>
      </template>
      <template v-else>
        <div class="status error" title="No Battery Present">:D</div>
      </template>
    </template>
  </Generic>
</template>

<script>
// Parts of the code are taken from Pihole, Ping, and OctoPrint service
import service from "@/mixins/service.js";

export default {
  name: "BatteryApi",
  mixins: [service],
  props: {
    item: Object,
  },
  data: () => ({
    present: true,
    // technology: null,
    // health: null,
    plugged: null,
    status: null,
    temperature: null,
    voltage: null,
    current: null,
    // currentAverage: null,
    percentage: 100,
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
            return "Full";
          case "OFFLINE":
            return "Offline";
          case "CHARGING":
            return "Charging";
          case "DISCHARGING":
            return "Using";
          case "NOT_CHARGING":
            return "Using";
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
        const response = await this.fetch("/");
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
    faDischargingIcon: function (percent) {
      if (percent > 90) {
        return "fa-battery-full";
      } else if (percent > 75) {
        return "fa-battery-three-quarters";
      } else if (percent > 40) {
        return "fa-battery-half";
      } else if (percent > 20) {
        return "fa-battery-quarter";
      } else {
        return "fa-battery-empty";
      }
    },
    faChargingIcon: function (plug) {
      if (plug == "PLUGGED_AC") {
        return "fa-plug-circle-bolt";
      } else if (plug == "PLUGGED_USB") {
        return "fa-computer";
      } else if (plug == "PLUGGED_DOCK") {
        return "fa-tv";
      } else if (plug == "PLUGGED_WIRELESS") {
        return "fa-wifi";
      } else if (plug == "UNPLUGGED") {
        return "fa-question";
      } else {
        return "fa-plug-circle-exclamation";
      }
    },
  },
};
</script>

<style scoped lang="scss">
.status {
  font-size: 0.8rem;
  color: var(--text-title);
}
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
