<script setup>
import { cn } from "@/lib/utils";

const props = defineProps({
  class: { type: null, required: false },
});
</script>

<template>
  <span
    data-slot="dropdown-menu-shortcut"
    :class="
      cn('text-muted-foreground ml-auto tracking-widest', props.class)
    "
  >
    <slot />
  </span>
</template>
