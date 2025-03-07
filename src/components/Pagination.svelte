<script lang="ts">
	import { applyAction } from '$app/forms';
	import { Button, Icon } from '$components';
	import { toastr } from '$components/toasts';
	import type { RequestSearchParams } from '$lib/request.svelte';

	let {
		url,
		prevPage,
		currentPage,
		nextPage,
		totalPages,
		collectionLength,
		totalRecords,
		searchParams,
		success = undefined
	}: {
		url: string;
		prevPage: null | number;
		currentPage: number;
		nextPage: null | number;
		totalPages: number;
		collectionLength: number;
		totalRecords: number;
		searchParams: RequestSearchParams;
		success?: (data: any) => Promise<void> | void;
	} = $props();

	async function onpagechange(page: number | null) {
		if (!page) return;

		searchParams.page = page;
		const response = await fetch(`${url}?${searchParams.toString()}`);

		if (response.ok) {
			return await success?.(await response.json());
		}

		if (response.status === 401) {
			const data = await response.json();

			if (data.message) {
				toastr.error(data.message);
			}

			return await applyAction({
				type: 'redirect',
				status: 303,
				location: data.location ?? '/'
			});
		}

		if (response.status === 303) {
			const data = await response.json();

			return await applyAction({
				type: 'redirect',
				status: 303,
				location: data.location
			});
		}
	}
</script>

<div class="inline-flex items-center justify-center gap-x-2">
	<Button
		color="light"
		size="sm"
		noborder
		title="First"
		class="h-8"
		onclick={() => onpagechange(1)}
		disabled={currentPage <= 1}
	>
		<Icon icon="chevrons-left" class="m-auto size-5" />
	</Button>
	<Button
		color="light"
		size="sm"
		noborder
		title="Prev"
		class="h-8"
		disabled={!prevPage}
		onclick={() => onpagechange(prevPage)}
	>
		<Icon icon="chevron-left" class="m-auto size-5" />
	</Button>
	<span class="text-sm text-gray-700 dark:text-gray-400">
		<span class="font-semibold text-gray-900 dark:text-white"
			>{!totalRecords ? 0 : searchParams.pageSize * (currentPage - 1) + 1}</span
		>
		to
		<span class="font-semibold text-gray-900 dark:text-white"
			>{!totalRecords
				? 0
				: searchParams.pageSize * (currentPage - 1) +
					Math.min(collectionLength, searchParams.pageSize)}</span
		>
		of <span class="font-semibold text-gray-900 dark:text-white">{totalRecords}</span>
	</span>
	<Button
		color="light"
		size="sm"
		noborder
		title="Next"
		class="h-8"
		onclick={() => onpagechange(nextPage)}
		disabled={!nextPage}
	>
		<Icon icon="chevron-right" class="m-auto size-5" />
	</Button>
	<Button
		color="light"
		size="sm"
		noborder
		title="Last"
		class="h-8"
		onclick={() => onpagechange(totalPages)}
		disabled={currentPage >= totalPages}
	>
		<Icon icon="chevrons-right" class="m-auto size-5" />
	</Button>
</div>
