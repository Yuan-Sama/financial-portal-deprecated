<script lang="ts">
	import { AppName } from '$lib';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<svelte:head>
	<title>{AppName} - Transactions</title>
</svelte:head>
