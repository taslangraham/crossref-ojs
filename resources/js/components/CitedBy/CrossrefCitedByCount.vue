<template>
	{{ store.total }}
</template>

<script setup>
import {useCrossrefCitedByStore} from './useCrossrefCitedByStore.js';

const props = defineProps({
	submissionId: {type: Number, required: true},
});

const store = useCrossrefCitedByStore();

store.initialize(props);
</script>
