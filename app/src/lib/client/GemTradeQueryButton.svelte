<script lang="ts">
	import type { GemProfitResponseItem } from '$lib/gemLevelProfitApi';
	import { poeTradeQueryResponseSchema, type PoeTradeGemRequest } from '$lib/pathOfExileApi';
	import { ProgressRadial } from '@skeletonlabs/skeleton';

	export let data: GemProfitResponseItem;
	export let name: string;

	let web_trade_url: Promise<string | undefined> = Promise.resolve(undefined);

	function startCreateTradeQuery() {
		web_trade_url = fetch('/api/pathOfExile/createTradeQuery', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				name,
				min_level: data.min.level,
				max_level: data.max.level,
				corrupted: false
			} satisfies PoeTradeGemRequest)
		}).then(async (res) => {
			if (res.status !== 200) {
				return undefined;
			}
			const rawResponse = await res.json();
			console.log(rawResponse);
			const response = poeTradeQueryResponseSchema.parse(rawResponse);
			return response.web_trade_url;
		});
	}
</script>

{#await web_trade_url}
	<button class="btn variant-soft-primary w-[5rem]" disabled>
		<ProgressRadial width="w-6" font={10} />
	</button>
{:then web_trade_url}
	{#if web_trade_url}
		<a class="btn variant-soft-success w-[5rem]" href={web_trade_url} target="_blank">Buy</a>
	{:else}
		<button class="btn variant-soft-primary w-[5rem]" on:click={startCreateTradeQuery}>Trade</button
		>
	{/if}
{:catch error}
	<button class="btn variant-soft-error w-[5rem]">Failed</button>
{/await}
