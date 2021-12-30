<script>
	import { fly, fade } from 'svelte/transition';
	export let onOk = () => {};
	export let onCancel = () => {};
	let value = '';
</script>

<div class="fixed z-10 inset-0 overflow-y-hidden" role="dialog" aria-modal="true">
	<div
		class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0 bg-black bg-opacity-60"
		in:fade={{ duration: 300 }}
		out:fade={{ duration: 300 }}
	>
		<span class="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true"
			>&#8203;</span
		>
		<div
			class="inline-block align-bottom bg-gray-800 rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle"
			in:fly={{ duration: 300, y: -1000 }}
			out:fly={{ duration: 300, y: -1000 }}
		>
			<div class="px-4 pt-5 pb-4 sm:p-6 sm:pb-8">
				<h2 class="text-xl font-bold">Passcode</h2>
				<span class="block text-slate-200 pb-2"> Herders, refer to Discord for the passcode. </span>
				<div class="grid grid-cols-2 auto-rows-auto gap-4">
					<label class="block col-span-2">
						<input
							bind:value
							type="password"
							class="
                    mt-1
                    block
                    w-full
                    rounded-md
                    border-gray-600 bg-slate-200
                    text-gray-900
                    shadow-sm
                    focus:outline-none
                  "
							placeholder=""
						/>
					</label>
					<div class="flex gap-2 row-start-2 col-start-2">
						<button
							on:click={onCancel}
							class="flex-1 flex items-center justify-center px-6 py-2 border border-transparent text-base font-medium rounded-md text-indigo-700 bg-indigo-100 hover:bg-indigo-200 md:py-2 md:text-lg md:px-6"
						>
							Cancel
						</button>
						<button
							class="flex-1 flex items-center justify-center px-6 py-2 border border-transparent text-base font-medium rounded-md text-indigo-100 bg-indigo-600 hover:bg-indigo-700 md:py-2 md:text-lg md:px-6"
							on:click={() => onOk(value)}
						>
							Export
						</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

<style>
	:global(html) {
		overflow-y: hidden;
	}
</style>
