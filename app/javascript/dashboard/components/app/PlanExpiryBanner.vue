<script setup>
import { computed } from 'vue';
import { format, parseISO } from 'date-fns';
import { useAccount } from 'dashboard/composables/useAccount';

const { currentAccount } = useAccount();

// Set per account in custom_attributes.plan_expires_on (YYYY-MM-DD).
// parseISO reads date-only strings as local time, so the day never shifts.
const expiryDate = computed(() => {
  const planExpiresOn =
    currentAccount.value?.custom_attributes?.plan_expires_on;
  return planExpiresOn ? format(parseISO(planExpiresOn), 'dd/MM') : null;
});
</script>

<!-- eslint-disable-next-line vue/no-root-v-if -->
<template>
  <div
    v-if="expiryDate"
    role="alert"
    class="flex items-center justify-center gap-2 px-4 py-2.5 text-sm font-medium text-white bg-n-ruby-9"
  >
    <span class="shrink-0 i-lucide-triangle-alert size-5" />
    <span>{{ $t('GENERAL_SETTINGS.PLAN_EXPIRES', { date: expiryDate }) }}</span>
  </div>
</template>
