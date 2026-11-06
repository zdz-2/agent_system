<template>
  <i-frame :src="url" />
</template>

<script setup>
import { computed } from "vue";
import iFrame from "@/components/iFrame/index.vue";
import { getToken } from "@/utils/auth";

defineOptions({
  name: "MesAgent",
});

const url = computed(() => {
  const baseUrl =
    import.meta.env.VITE_APP_LUXFLOW_ERP_URL || "http://127.0.0.1:31300";
  const token = getToken();

  if (!token) {
    return baseUrl;
  }

  const separator = baseUrl.includes("?") ? "&" : "?";
  return `${baseUrl}${separator}token=${encodeURIComponent(token)}`;
});
</script>
