<script>
	import { fade } from 'svelte/transition';
	import Modal from '$lib/components/modal.svelte';
	import { page } from '$app/stores';
	import { onMount } from 'svelte';
	import Editor from '$lib/components/editor.svelte';
	import Prompt from '$lib/components/prompt.svelte';

	let modal = { show: false };
	let editor = { show: false };
	let prompt = { show: false };

	let modify = false;
	let author = '';
	let id = '';
	let loading = true;
	let status = 'Loading...';
	let filter = '';

	const fail = (message, newStatus) => {
		modal = {
			show: true,
			title: 'Error',
			message: message,
			onOk: () => {
				modal.show = false;
				status = newStatus;
			}
		};
	};

	let urls = {};
	let sorted = [];
	$: {
		sorted = Object.keys(urls).sort((a, b) => {
			const left = urls[a].keywords[0].toLowerCase(),
				right = urls[b].keywords[0].toLowerCase();
			return left < right ? -1 : 1;
		});
		if (filter !== '') {
			sorted = sorted.filter((url) => {
				const item = urls[url];
				if (
					url.includes(filter) ||
					item.title.includes(filter) ||
					item.keywords.some((i) => i.includes(filter))
				) {
					return true;
				}
				return false;
			});
		}
	}

	const create = () => {
		editor = {
			author: author,
			id: id,
			show: true,
			onOk: (url, title, keywords, author, id) => {
				editor.show = false;
				urls[url] = {
					url,
					keywords: keywords
						.split('\n')
						.map((s) => s.trim())
						.filter((s) => s !== ''),
					title,
					added: new Date().toISOString(),
					author,
					id
				};
			},
			onCancel: () => {
				editor.show = false;
			}
		};
	};

	export const exp = () => {
		prompt = {
			show: true,
			onOk: (code) => {
				prompt.show = false;
				const data = new FormData();
				data.append('pass', code);
				data.append('file', new Blob([JSON.stringify(urls)]), 'payload.json');
				fetch('https://f.teamortix.com/upload', {
					method: 'POST',
					body: data
				})
					.then((r) => r.text())
					.then((text) => {
						text = text.replace(/https:\/\/f.teamortix.com\/hosted\//, '');
						text = text.replace(/\.json/, '');
						modal = {
							show: true,
							ok: true,
							title: 'Success: ' + text,
							message: '/dgo apply code:' + text,
							onOk: () => {
								modal.show = false;
							}
						};
					})
					.catch(() => {
						console.error('could not export: %s', e.message);
						fail('Could not export: Is your code invalid? Check console for more details', '');
					});
			},
			onCancel: () => {
				prompt.show = false;
			}
		};
	};

	const edit = (url) => {
		const item = urls[url];
		editor = {
			...item,
			show: true,
			keywords: item.keywords.join('\n'),
			onOk: (url, title, keywords, author, id) => {
				editor.show = false;
				delete urls[item.url];
				urls[url] = {
					url,
					keywords: keywords
						.split('\n')
						.map((s) => s.trim())
						.filter((s) => s !== ''),
					title,
					added: new Date().toISOString(),
					author,
					id
				};
			},
			onCancel: () => {
				editor.show = false;
			}
		};
	};

	const rem = (url) => {
		modal = {
			show: true,
			title: 'Confirm Deletion',
			message: `URL: ${url}\n\n`,
			onOk: () => {
				modal.show = false;
				delete urls[url];
				urls = urls;
			},
			onCancel: () => {
				modal.show = false;
			}
		};
	};

	const code = $page.params.code;
	onMount(async () => {
		fetch(`https://f.teamortix.com/hosted/${code}.json`)
			.then((res) => res.json())
			.then((data) => {
				loading = false;
				if (data.type === 'edit') {
					modify = true;
					author = data.author;
					id = data.id;
					urls = data.urls;
				} else {
					urls = data;
				}
			})
			.catch((e) => {
				console.error('could not load data: %s', e.message);
				fail(
					'Could not load data. Is your code invalid? Check console for more details',
					'Not found :('
				);
			});
	});
</script>

{#if modal.show}
	<Modal {...modal} />
{/if}

{#if editor.show}
	<Editor {...editor} />
{/if}

{#if prompt.show}
	<Prompt {...prompt} />
{/if}

{#if loading}
	<div class="flex my-16 max-w-full justify-center ">
		<h1 class="text-3xl tracking-tight font-extrabold text-indigo-100 sm:text-4xl md:text-5xl">
			<span class="block">{status}</span>
		</h1>
	</div>
{:else}
	<div class="flex flex-col max-w-full mx-8 md:mx-16 my-16 gap-12" in:fade out:fade>
		<div class="mt-5 sm:mt-8 sm:flex sm:justify-center md:justify-start">
			{#if modify}
				<div class="rounded-md shadow">
					<button
						on:click={create}
						class="w-full flex items-center justify-center px-6 py-2 border border-transparent text-base font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 md:py-3 md:text-lg md:px-6"
					>
						New
					</button>
				</div>
				<div class="mt-3 sm:mt-0 sm:ml-3">
					<button
						on:click={exp}
						class="w-full flex items-center justify-center px-6 py-2 border border-transparent text-base font-medium rounded-md text-indigo-700 bg-indigo-100 hover:bg-indigo-200 md:py-3 md:text-lg md:px-6"
					>
						Export
					</button>
				</div>
			{/if}
			<input
				class="mt-3 sm:mt-0 sm:ml-3 rounded-md shadow-sm focus:outline-none max-w-xl w-full text-base font-medium text-indigo-700"
				type="text"
				bind:value={filter}
				placeholder="Filter"
			/>
		</div>

		<div class="-my-2 overflow-x-auto sm:-mx-6 lg:-mx-8">
			<div class="py-2 align-middle inline-block min-w-full sm:px-6 lg:px-8">
				<div class="shadow overflow-hidden border-b border-gray-600 sm:rounded-lg">
					<table class="min-w-full divide-y divide-gray-600">
						<thead class="bg-indigo-600">
							<tr>
								<th
									scope="col"
									class="py-3 pl-7 text-left text-xs font-medium text-white uppercase tracking-wider"
								>
									{#if modify} Edit {:else} Author {/if}
								</th>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium text-white uppercase tracking-wider"
								>
									Keywords
								</th>
								<th
									scope="col"
									class="px-6 py-3 text-left text-xs font-medium text-white uppercase tracking-wider"
								>
									<span class="font-bold">Title</span>
									<br />
									<span class="">URL</span>
								</th>
							</tr>
						</thead>
						<tbody class="bg-gray-800 divide-y divide-gray-600">
							{#each sorted as s}
								<tr class="hover:bg-gray-700">
									<td class="whitespace-nowrap text-left pl-5 py-3">
										{#if modify}
											<span class="inline-flex gap-1" on:click={() => edit(s)}>
												<svg
													xmlns="http://www.w3.org/2000/svg"
													class="h-4 w-4 hover:text-indigo-200"
													fill="none"
													viewBox="0 0 24 24"
													stroke="currentColor"
												>
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														stroke-width="2"
														d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"
													/>
												</svg>
											</span>
											<span class="inline-flex gap-1" on:click={() => rem(s)}>
												<svg
													xmlns="http://www.w3.org/2000/svg"
													class="h-4 w-6 text-red-400 hover:text-red-500"
													fill="none"
													viewBox="0 0 24 24"
													stroke="currentColor"
												>
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														stroke-width="2"
														d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
													/>
												</svg>
											</span>
										{:else}
											<div class="text-sm text-gray-200">{urls[s].author}</div>
										{/if}
									</td>
									<td class="px-6 py-4 whitespace-nowrap">
										{#each urls[s].keywords as k}
											<div class="text-sm text-gray-200">{k}</div>
										{/each}
									</td>
									<td class="px-6 py-4 whitespace-nowrap">
										<div class="flex items-center">
											<div class="text-sm font-medium text-white">
												<span class="font-bold">{urls[s].title}</span>
												<br />
												<a href={s}>{s}</a>
											</div>
										</div>
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>
		</div>
	</div>
{/if}

<style>
</style>
