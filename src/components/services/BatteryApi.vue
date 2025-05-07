<template>
  <Generic :item="item">
    <template #icon>
      <!-- left area containing the icon -->
    </template>
    <template #content>
      <!-- main area containing the title, subtitle, ... -->
    </template>
    <template #indicator>
      <!-- top right area, empty by default -->
    </template>
  </Generic>
</template>

<script>

export default {
  name: "BatteryApi",
  props: {
    item: Object,
  },
};
</script>
