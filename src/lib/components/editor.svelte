<script>
	import Modal from './modal.svelte';
	import { fly, fade } from 'svelte/transition';
	export let url, title, keywords, author, id;
	export let onOk = () => {};
	export let onCancel = () => {};

	let modal = { show: false };
	const fail = (message) => {
		modal = {
			show: true,
			title: 'Error',
			message: message,
			onOk: () => (modal.show = false)
		};
	};

	const submit = () => {
		try {
			if (title === '') throw new Error('Title is required');
			new URL(url);
			if (keywords.trim() === '') throw new Error('Keywords is required (one per line)');
			if (author === '') throw new Error('Author is required');
			if (id === '') throw new Error('ID is required');
			onOk(url, title, keywords.trim(), author, id);
		} catch (e) {
			fail(e.message);
		}
	};
</script>

{#if modal.show}
	<div class="z-20">
		<Modal {...modal} />
	</div>
{/if}

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
				<h2 class="text-xl font-bold mb-2">Edit Resource</h2>
				<div class="grid grid-cols-2 auto-rows-auto gap-4">
					<label class="block">
						<span class="text-slate-200">Title</span>
						<input
							type="text"
							bind:value={title}
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
					<label class="block">
						<span class="text-slate-200">URL</span>
						<input
							type="text"
							bind:value={url}
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
					<label class="col-span-2">
						<span class="text-slate-200">Keywords</span>
						<textarea
							bind:value={keywords}
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
							rows="3"
						/>
					</label>
					<label class="block">
						<span class="text-slate-200">Author</span>
						<input
							type="text"
							bind:value={author}
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
					<label class="block">
						<span class="text-slate-200">Author ID</span>
						<input
							type="text"
							bind:value={id}
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
					<div class="flex gap-2 col-start-2">
						<button
							on:click={onCancel}
							class="flex-1 flex items-center justify-center px-6 py-2 border border-transparent text-base font-medium rounded-md text-indigo-700 bg-indigo-100 hover:bg-indigo-200 md:py-2 md:text-lg md:px-6"
						>
							Cancel
						</button>
						<button
							on:click={submit}
							class="flex-1 flex items-center justify-center px-6 py-2 border border-transparent text-base font-medium rounded-md text-indigo-100 bg-indigo-600 hover:bg-indigo-700 md:py-2 md:text-lg md:px-6"
						>
							Save
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
