<script lang="ts">
	import { onMount } from 'svelte';
	import axios from 'axios';
	import { Button } from '$lib/components/ui/button/index.js';
	import * as Command from '$lib/components/ui/command/index.js';
	import * as Dialog from '$lib/components/ui/dialog/index.js';
	import * as Sheet from '$lib/components/ui/sheet/index.js';
	import * as Table from '$lib/components/ui/table/index.js';

	let items: any[] = [];
	let page = 1;
	const pageSize = 25;
	let total = 0;
	let totalPages = 1;
	let toPage = 1;

	let autocomplete_authors: any[] = [];
	const id = 'i190243414'; // OpenAlex institution ID of President University (https://openalex.org/institutions/i190243414)

	let searchTerm = '';
	let authorId = '';
	let authorName = '';
	let ORCID_iD = '';

	const getAuthors = async () => {
		try {
			let api = `https://api.openalex.org/autocomplete/authors?filter=last_known_institutions.id:${id}`;

			if (authorName.trim() !== '') {
				api += `&q=${encodeURIComponent(authorName.trim())}`;
			}

			console.log(api);
			const response = await axios.get(api);
			autocomplete_authors = response.data.results;
			console.log('getAuthors successful', autocomplete_authors);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getAuthors:', error.message);
		}
	};

	onMount(getAuthors);

	const fetchData = async () => {
		try {
			let api = `https://api.openalex.org/authors?page=${page}&per-page=${pageSize}&sort=works_count:desc&filter=last_known_institutions.id:${id}`;

			if (searchTerm.trim() !== '') {
				api += `,default.search:${encodeURIComponent(searchTerm.trim())}`;
			}

			if (authorId.trim() !== '') {
				api += `,ids.openalex:${encodeURIComponent(authorId.trim())}`;
			}

			if (ORCID_iD !== '') {
				api += `,has_orcid:true`;
			}

			console.log(api);
			const response = await axios.get(api);
			items = response.data.results;
			console.log('fetchData successful', items);
			console.log(response.data);
			console.log(response.data.meta.count, 'results');

			total = response.data.meta.count;
			totalPages = Math.ceil(total / pageSize);
		} catch (error: any) {
			console.error('There was a problem with the GET request:', error.message);
		}
	};

	onMount(fetchData);

	async function nextPage() {
		page += 1;
		toPage += 1;
		await fetchData();
	}

	async function prevPage() {
		page -= 1;
		toPage -= 1;
		await fetchData();
	}

	async function jumpPage() {
		if (toPage > totalPages) {
			console.log('Exceeds maximum page');
			toPage = page;
		} else if (toPage < 1) {
			console.log('Exceeds minimum page');
			toPage = page;
		} else {
			page = toPage;
			await fetchData();
		}
	}

	const autocompleteAuthor = async (event: any) => {
		authorName = event.target.value;
		await getAuthors();
	};

	function runSearch() {
		page = 1;
		toPage = 1;
		authorId = '';
		authorName = '';
		fetchData();
	}

	function clearSearch() {
		page = 1;
		toPage = 1;
		searchTerm = '';
		authorId = '';
		authorName = '';
		ORCID_iD = '';
		fetchData();
	}

	function clearAuthor() {
		page = 1;
		toPage = 1;
		searchTerm = '';
		authorId = '';
		authorName = '';
		fetchData();
	}

	function copyAuthorId(id: string) {
		navigator.clipboard
			.writeText(id)
			.then(() => {
				authorId = id;
				searchTerm = '';
				page = 1;
				toPage = 1;
				fetchData();
			})
			.catch((error) => {
				console.error('Failed to copy:', error);
			});
	}

	function copyAuthorName(name: string) {
		navigator.clipboard;
		authorName = name;
	}

	function has_ORCID_iD() {
		ORCID_iD = 'y';
		page = 1;
		toPage = 1;
		fetchData();
	}

	function reset_identifier() {
		ORCID_iD = '';
		page = 1;
		toPage = 1;
		fetchData();
	}
</script>

<div class="flex-1">
	<div class="border-b">
		<div class="mx-auto w-full min-w-0 max-w-screen-2xl" style="padding: 2rem;">
			<div class="space-y-2">
				<h1 class="scroll-m-20 text-4xl font-bold tracking-tight">Authors</h1>
				<p class="text-balance text-lg text-muted-foreground">People who create works.</p>
			</div>
			<div id="markdown" class="markdown pb-12 pt-8">
				<div class="group relative my-4 flex flex-col space-y-2">
					<div class="relative mr-auto w-full">
						<div
							class="relative mt-2 rounded-md border ring-offset-background focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2"
						>
							<div class="theme-zinc w-full">
								<div class="preview flex min-h-[350px] w-full items-center justify-center p-10">
									<div class="relative w-full overflow-auto">
										<div class="flex items-center py-4">
											<div
												class="flex flex-1 items-center justify-between space-x-2 md:justify-start"
											>
												<Dialog.Root>
													<div class="w-full flex-1 md:w-auto md:flex-none">
														<Dialog.Trigger>
															<Button size="icon" variant="outline">
																<svg
																	width="15"
																	height="15"
																	viewBox="0 0 15 15"
																	fill="none"
																	xmlns="http://www.w3.org/2000/svg"
																	><path
																		d="M7.5 0.875C5.49797 0.875 3.875 2.49797 3.875 4.5C3.875 6.15288 4.98124 7.54738 6.49373 7.98351C5.2997 8.12901 4.27557 8.55134 3.50407 9.31167C2.52216 10.2794 2.02502 11.72 2.02502 13.5999C2.02502 13.8623 2.23769 14.0749 2.50002 14.0749C2.76236 14.0749 2.97502 13.8623 2.97502 13.5999C2.97502 11.8799 3.42786 10.7206 4.17091 9.9883C4.91536 9.25463 6.02674 8.87499 7.49995 8.87499C8.97317 8.87499 10.0846 9.25463 10.8291 9.98831C11.5721 10.7206 12.025 11.8799 12.025 13.5999C12.025 13.8623 12.2376 14.0749 12.5 14.0749C12.7623 14.075 12.975 13.8623 12.975 13.6C12.975 11.72 12.4778 10.2794 11.4959 9.31166C10.7244 8.55135 9.70025 8.12903 8.50625 7.98352C10.0187 7.5474 11.125 6.15289 11.125 4.5C11.125 2.49797 9.50203 0.875 7.5 0.875ZM4.825 4.5C4.825 3.02264 6.02264 1.825 7.5 1.825C8.97736 1.825 10.175 3.02264 10.175 4.5C10.175 5.97736 8.97736 7.175 7.5 7.175C6.02264 7.175 4.825 5.97736 4.825 4.5Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
															</Button>
														</Dialog.Trigger>
														<Dialog.Content class="padding p-0">
															<Command.Root>
																<div class="flex items-center border-b px-3">
																	<svg
																		class="mr-2 h-4 w-4 shrink-0 opacity-50"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M10 6.5C10 8.433 8.433 10 6.5 10C4.567 10 3 8.433 3 6.5C3 4.567 4.567 3 6.5 3C8.433 3 10 4.567 10 6.5ZM9.30884 10.0159C8.53901 10.6318 7.56251 11 6.5 11C4.01472 11 2 8.98528 2 6.5C2 4.01472 4.01472 2 6.5 2C8.98528 2 11 4.01472 11 6.5C11 7.56251 10.6318 8.53901 10.0159 9.30884L12.8536 12.1464C13.0488 12.3417 13.0488 12.6583 12.8536 12.8536C12.6583 13.0488 12.3417 13.0488 12.1464 12.8536L9.30884 10.0159Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	<input
																		class="flex h-11 w-full rounded-md bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
																		type="text"
																		bind:value={authorName}
																		on:input={autocompleteAuthor}
																		placeholder="Type author name"
																	/>
																</div>
																{#if authorId !== ''}
																	<Command.Group heading="Actions">
																		<Command.Item>
																			<button
																				class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																				on:click={clearAuthor}
																			>
																				<p style="display: inline-flex; align-items:center">
																					<svg
																						class="margin-right mr-2"
																						width="15"
																						height="15"
																						viewBox="0 0 15 15"
																						fill="none"
																						xmlns="http://www.w3.org/2000/svg"
																						><path
																							d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																							fill="currentColor"
																							fill-rule="evenodd"
																							clip-rule="evenodd"
																						></path></svg
																					>
																					Reset
																				</p>
																			</button>
																		</Command.Item>
																	</Command.Group>
																{:else}
																	<div></div>
																{/if}
																<Command.List>
																	<Command.Group heading="Authors">
																		<Command.Empty>No results found.</Command.Empty>
																		{#each autocomplete_authors as author}
																			<Command.Item>
																				<button
																					class="padding p-.5 relative flex cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() =>
																						copyAuthorId(
																							author.id.substring('https://openalex.org/'.length)
																						)}
																					on:click={() => copyAuthorName(author.display_name)}
																				>
																					{author.display_name}
																				</button>
																			</Command.Item>
																		{/each}
																	</Command.Group>
																	<Command.Separator />
																</Command.List>
																{#if authorId !== ''}
																	<div
																		class="
																padding relative flex cursor-default select-none items-center
																rounded-sm p-2 text-sm outline-none hover:text-blue-500
																aria-selected:bg-accent aria-selected:text-accent-foreground
																data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																	>
																		<p
																			style="display: inline-flex; align-items:center"
																			class="text-sm dark:text-lime-400"
																		>
																			<svg
																				class="margin-right mr-2"
																				width="15"
																				height="15"
																				viewBox="0 0 15 15"
																				fill="none"
																				xmlns="http://www.w3.org/2000/svg"
																				><path
																					d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																					fill="currentColor"
																					fill-rule="evenodd"
																					clip-rule="evenodd"
																				></path></svg
																			>
																			Filter active: {authorName}
																		</p>
																	</div>
																{:else}
																	<div></div>
																{/if}
															</Command.Root>
														</Dialog.Content>
													</div>
												</Dialog.Root>

												<div class="w-full flex-1 md:w-auto md:flex-none">
													<form on:submit={runSearch}>
														<input
															class="flex h-9 w-screen max-w-sm rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 only:focus-visible:outline-none disabled:cursor-not-allowed disabled:opacity-50"
															type="text"
															bind:value={searchTerm}
															placeholder="Name"
														/>
													</form>
												</div>
												<Button variant="default" size="icon" on:click={runSearch}>
													<svg
														xmlns="http://www.w3.org/2000/svg"
														viewBox="0 0 24 24"
														fill="none"
														stroke="currentColor"
														stroke-width="2"
														stroke-linecap="round"
														stroke-linejoin="round"
														class="lucide lucide-search h-[1.2rem] w-[1.2rem]"
														><circle cx="11" cy="11" r="8" /><path d="m21 21-4.3-4.3" /></svg
													>
												</Button>
												{#if authorId !== '' || searchTerm == ''}
													<Button size="icon" style="visibility: hidden"></Button>
												{:else}
													<Button variant="ghost" size="icon" on:click={clearAuthor}>
														<svg
															width="15"
															height="15"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M12.8536 2.85355C13.0488 2.65829 13.0488 2.34171 12.8536 2.14645C12.6583 1.95118 12.3417 1.95118 12.1464 2.14645L7.5 6.79289L2.85355 2.14645C2.65829 1.95118 2.34171 1.95118 2.14645 2.14645C1.95118 2.34171 1.95118 2.65829 2.14645 2.85355L6.79289 7.5L2.14645 12.1464C1.95118 12.3417 1.95118 12.6583 2.14645 12.8536C2.34171 13.0488 2.65829 13.0488 2.85355 12.8536L7.5 8.20711L12.1464 12.8536C12.3417 13.0488 12.6583 13.0488 12.8536 12.8536C13.0488 12.6583 13.0488 12.3417 12.8536 12.1464L8.20711 7.5L12.8536 2.85355Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>
													</Button>
												{/if}
												{#if toPage > totalPages}
													<p
														style="display: inline-flex; align-items:center"
														class="text-sm text-orange-700 dark:text-orange-400"
													>
														<svg
															xmlns="http://www.w3.org/2000/svg"
															viewBox="0 0 24 24"
															fill="none"
															stroke="currentColor"
															stroke-width="2"
															stroke-linecap="round"
															stroke-linejoin="round"
															class="lucide lucide-ban mr-2 h-[1rem] w-[1rem]"
															><circle cx="12" cy="12" r="10" /><path d="m4.9 4.9 14.2 14.2" /></svg
														>Above maximum page limit
													</p>
												{:else if toPage < 1}
													<p
														style="display: inline-flex; align-items:center"
														class="text-sm text-orange-700 dark:text-orange-400"
													>
														<svg
															xmlns="http://www.w3.org/2000/svg"
															viewBox="0 0 24 24"
															fill="none"
															stroke="currentColor"
															stroke-width="2"
															stroke-linecap="round"
															stroke-linejoin="round"
															class="lucide lucide-ban mr-2 h-[1rem] w-[1rem]"
															><circle cx="12" cy="12" r="10" /><path d="m4.9 4.9 14.2 14.2" /></svg
														>Below minimum page limit
													</p>
												{:else}
													<div></div>
												{/if}
											</div>
											<div
												class="ml-auto inline-flex h-10 items-center justify-between space-x-2 whitespace-nowrap rounded-md bg-background px-4 py-2 text-sm font-medium ring-offset-background"
											>
												<t class="text-muted-foreground">
													Page {page} of {totalPages}
												</t>
												{#if page == 1}
													<Button
														style="visibility: hidden;"
														variant="ghost"
														size="icon"
														on:click={prevPage}
													>
														prev
													</Button>
												{:else}
													<Button variant="ghost" size="icon" on:click={prevPage}>
														<svg
															xmlns="http://www.w3.org/2000/svg"
															width="24"
															height="24"
															viewBox="0 0 24 24"
															fill="none"
															stroke="currentColor"
															stroke-width="2"
															stroke-linecap="round"
															stroke-linejoin="round"
															class="lucide lucide-chevron-left"><path d="m15 18-6-6 6-6" /></svg
														>
													</Button>
												{/if}
												<!-- <div id="separator"></div> -->
												<!-- {#if items.length < pageSize} -->
												{#if page == totalPages}
													<Button
														style="visibility: hidden"
														variant="ghost"
														size="icon"
														on:click={nextPage}
													>
														next
													</Button>
												{:else}
													<Button variant="ghost" size="icon" on:click={nextPage}>
														<svg
															xmlns="http://www.w3.org/2000/svg"
															width="24"
															height="24"
															viewBox="0 0 24 24"
															fill="none"
															stroke="currentColor"
															stroke-width="2"
															stroke-linecap="round"
															stroke-linejoin="round"
															class="lucide lucide-chevron-right"><path d="m9 18 6-6-6-6" /></svg
														>
													</Button>
												{/if}
												<form on:submit={jumpPage}>
													<input
														class="flex h-9 w-20 max-w-sm rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50"
														type="number"
														min="1"
														max={totalPages}
														bind:value={toPage}
													/>
												</form>
												<Button variant="ghost" size="icon" on:click={jumpPage}>
													<svg
														xmlns="http://www.w3.org/2000/svg"
														width="24"
														height="24"
														viewBox="0 0 24 24"
														fill="none"
														stroke="currentColor"
														stroke-width="2"
														stroke-linecap="round"
														stroke-linejoin="round"
														class="lucide lucide-chevrons-right"
														><path d="m6 17 5-5-5-5" /><path d="m13 17 5-5-5-5" /></svg
													>
												</Button>
											</div>
										</div>
										<div class="flex items-center py-4">
											<div
												class="flex flex-1 items-center justify-between space-x-2 md:justify-start"
											>
												<Button variant="destructive" size="sm" on:click={clearSearch}>
													<svg
														width="15"
														height="15"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
												{#if ORCID_iD !== ''}
													<Button size="sm" variant="ghost" on:click={reset_identifier}>
														<svg
															class="margin-right mr-2 dark:text-lime-400"
															width="15"
															height="15"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>Has ORCID iD
													</Button>
												{:else}
													<Button
														size="sm"
														variant="ghost"
														on:click={has_ORCID_iD}
														class="text text-muted-foreground"
													>
														<svg
															class="margin-right mr-2"
															width="15"
															height="15"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M0.877075 7.49991C0.877075 3.84222 3.84222 0.877075 7.49991 0.877075C11.1576 0.877075 14.1227 3.84222 14.1227 7.49991C14.1227 11.1576 11.1576 14.1227 7.49991 14.1227C3.84222 14.1227 0.877075 11.1576 0.877075 7.49991ZM7.49991 1.82708C4.36689 1.82708 1.82708 4.36689 1.82708 7.49991C1.82708 10.6329 4.36689 13.1727 7.49991 13.1727C10.6329 13.1727 13.1727 10.6329 13.1727 7.49991C13.1727 4.36689 10.6329 1.82708 7.49991 1.82708Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>Has ORCID iD
													</Button>
												{/if}

												<t class="text-sm font-medium text-muted-foreground">
													<!-- <span class="font-bold">
														{total}
													</span>
													{#if total > 1 || total == 0}
														results
													{:else}
														result
													{/if} -->
													{#if authorId !== ''}
														<Button
															variant="ghost"
															size="sm"
															class="text text-md"
															on:click={clearAuthor}
														>
															{authorName}
															<svg
																class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
																viewBox="0 0 15 15"
																fill="none"
																xmlns="http://www.w3.org/2000/svg"
																><path
																	d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
																	fill="currentColor"
																	fill-rule="evenodd"
																	clip-rule="evenodd"
																></path></svg
															>
														</Button>
													{:else}
														<div></div>
													{/if}
												</t>
											</div>
										</div>
										<Table.Root>
											<Table.Caption>
												<div
													style="display: flex; justify-content: space-between; align-items: baseline"
												>
													{#if page == 1}
														<div style="visibility: hidden;">
															<Button variant="ghost" on:click={prevPage}>
																<svg
																	xmlns="http://www.w3.org/2000/svg"
																	width="24"
																	height="24"
																	viewBox="0 0 24 24"
																	fill="none"
																	stroke="currentColor"
																	stroke-width="2"
																	stroke-linecap="round"
																	stroke-linejoin="round"
																	class="lucide lucide-chevron-left"
																	><path d="m15 18-6-6 6-6" /></svg
																>
															</Button>
														</div>
													{:else}
														<div>
															<Button variant="ghost" on:click={prevPage}>
																<svg
																	xmlns="http://www.w3.org/2000/svg"
																	width="24"
																	height="24"
																	viewBox="0 0 24 24"
																	fill="none"
																	stroke="currentColor"
																	stroke-width="2"
																	stroke-linecap="round"
																	stroke-linejoin="round"
																	class="lucide lucide-chevron-left"
																	><path d="m15 18-6-6 6-6" /></svg
																>
															</Button>
														</div>
													{/if}
													<!-- <div id="separator"></div> -->
													<div>
														Page {page} of {totalPages}
													</div>
													{#if items.length < pageSize}
														<div style="visibility: hidden;">
															<Button variant="ghost" on:click={nextPage}>next</Button>
														</div>
													{:else}
														<div>
															<Button variant="ghost" on:click={nextPage}>
																<svg
																	xmlns="http://www.w3.org/2000/svg"
																	width="24"
																	height="24"
																	viewBox="0 0 24 24"
																	fill="none"
																	stroke="currentColor"
																	stroke-width="2"
																	stroke-linecap="round"
																	stroke-linejoin="round"
																	class="lucide lucide-chevron-right"
																	><path d="m9 18 6-6-6-6" /></svg
																>
															</Button>
														</div>
													{/if}
												</div>
											</Table.Caption>
											<Table.Footer>
												<Table.Row></Table.Row>
											</Table.Footer>
											<Table.Header>
												<Table.Row>
													<Table.Head>
														<span class="font-bold">
															{total}
														</span>
														{#if total > 1 || total == 0}
															authors
														{:else}
															author
														{/if}
													</Table.Head>
													<Table.Head class="text-right">Has ORCID iD</Table.Head>
												</Table.Row>
											</Table.Header>
											<Table.Body>
												{#each items as work}
													<Table.Row>
														<Table.Cell class="font-medium">
															<a href={work.id} target="_blank">
																{work.display_name}
															</a>
															<div style="display: flex; " class="text-sm text-muted-foreground">
																<div>
																	{#each work.last_known_institutions as last_known}
																		<Button
																			variant="ghost"
																			size="sm"
																			class="text text-md"
																			href={last_known.id}
																			target="_blank"
																		>
																			{last_known.display_name}
																		</Button>
																	{/each}
																</div>
															</div>
															<div style="display: flex; ">
																{#if work.works_count > 0}
																	<div>
																		<Button
																			class="font-semibold"
																			variant="ghost"
																			size="sm"
																			target="_blank"
																			rel="noreferrer noopenner"
																			href="https://openalex.org/works?page=1&filter=authorships.author.id:{work.id.substring(
																				'https://openalex.org/'.length
																			)}"
																		>
																			{work.works_count} works
																		</Button>
																	</div>
																{:else}
																	<div></div>
																{/if}
															</div>
														</Table.Cell>
														<Table.Cell class="text-right">
															<div>
																<Sheet.Root>
																	<Sheet.Trigger asChild let:builder>
																		{#if work.orcid !== null}
																			<Button
																				builders={[builder]}
																				variant="ghost"
																				size="sm"
																				class="text text-md"
																			>
																				<svg
																					xmlns="http://www.w3.org/2000/svg"
																					viewBox="0 0 24 24"
																					fill="none"
																					stroke="currentColor"
																					stroke-width="2"
																					stroke-linecap="round"
																					stroke-linejoin="round"
																					class="lucide lucide-diamond mr-2 h-[1rem] w-[1rem] dark:stroke-lime-400"
																					><path
																						d="M2.7 10.3a2.41 2.41 0 0 0 0 3.41l7.59 7.59a2.41 2.41 0 0 0 3.41 0l7.59-7.59a2.41 2.41 0 0 0 0-3.41l-7.59-7.59a2.41 2.41 0 0 0-3.41 0Z"
																					/></svg
																				>
																				YES
																				<svg
																					xmlns="http://www.w3.org/2000/svg"
																					width="24"
																					height="24"
																					viewBox="0 0 24 24"
																					fill="none"
																					stroke="currentColor"
																					stroke-width="2"
																					stroke-linecap="round"
																					stroke-linejoin="round"
																					class="lucide lucide-chevron-right margin-left ml-2"
																					><path d="m9 18 6-6-6-6" /></svg
																				>
																			</Button>
																		{:else}
																			<Button
																				builders={[builder]}
																				variant="ghost"
																				size="sm"
																				class="text text-md"
																			>
																				<t class="text text-muted-foreground"> NO </t>
																				<svg
																					xmlns="http://www.w3.org/2000/svg"
																					width="24"
																					height="24"
																					viewBox="0 0 24 24"
																					fill="none"
																					stroke="currentColor"
																					stroke-width="2"
																					stroke-linecap="round"
																					stroke-linejoin="round"
																					class="lucide lucide-chevron-right margin-left ml-2"
																					><path d="m9 18 6-6-6-6" /></svg
																				>
																			</Button>
																		{/if}
																	</Sheet.Trigger>
																	<Sheet.Content
																		side="right"
																		class="h-screen max-h-max overflow-y-auto"
																	>
																		<Sheet.Header>
																			<Sheet.Title>{work.display_name}</Sheet.Title>
																			<Sheet.Description style="text-transform: capitalize">
																				<p style="display: inline-flex; align-items:center">
																					<svg
																						class="margin-right mr-2 h-[1rem] w-[1rem]"
																						viewBox="0 0 15 15"
																						fill="none"
																						xmlns="http://www.w3.org/2000/svg"
																						><path
																							d="M7.5 0.875C5.49797 0.875 3.875 2.49797 3.875 4.5C3.875 6.15288 4.98124 7.54738 6.49373 7.98351C5.2997 8.12901 4.27557 8.55134 3.50407 9.31167C2.52216 10.2794 2.02502 11.72 2.02502 13.5999C2.02502 13.8623 2.23769 14.0749 2.50002 14.0749C2.76236 14.0749 2.97502 13.8623 2.97502 13.5999C2.97502 11.8799 3.42786 10.7206 4.17091 9.9883C4.91536 9.25463 6.02674 8.87499 7.49995 8.87499C8.97317 8.87499 10.0846 9.25463 10.8291 9.98831C11.5721 10.7206 12.025 11.8799 12.025 13.5999C12.025 13.8623 12.2376 14.0749 12.5 14.0749C12.7623 14.075 12.975 13.8623 12.975 13.6C12.975 11.72 12.4778 10.2794 11.4959 9.31166C10.7244 8.55135 9.70025 8.12903 8.50625 7.98352C10.0187 7.5474 11.125 6.15289 11.125 4.5C11.125 2.49797 9.50203 0.875 7.5 0.875ZM4.825 4.5C4.825 3.02264 6.02264 1.825 7.5 1.825C8.97736 1.825 10.175 3.02264 10.175 4.5C10.175 5.97736 8.97736 7.175 7.5 7.175C6.02264 7.175 4.825 5.97736 4.825 4.5Z"
																							fill="currentColor"
																							fill-rule="evenodd"
																							clip-rule="evenodd"
																						></path></svg
																					>
																					Author
																				</p>
																			</Sheet.Description>
																			<div style="display:inline-flex; ">
																				<div>
																					<Button
																						variant="default"
																						size="sm"
																						class="text text-md"
																						href="https://openalex.org/works?page=1&filter=authorships.author.id:{work.id.substring(
																							'https://openalex.org/'.length
																						)}"
																						target="_blank"
																						rel="noreferrer noopenner"
																					>
																						View works
																					</Button>
																					<Button
																						variant="ghost"
																						class="text text-md font-mono font-semibold"
																						href="https://api.openalex.org/{work.id.substring(
																							'https://openalex.org/'.length
																						)}"
																						target="_blank"
																						rel="noreferrer noopenner"
																					>
																						API
																					</Button>
																					<Button
																						variant="ghost"
																						size="sm"
																						class="text text-md"
																						href={work.id}
																						target="_blank"
																						rel="noreferrer noopenner"
																						>OpenAlex
																						<svg
																							class="margin-left ml-2"
																							width="15"
																							height="15"
																							viewBox="0 0 15 15"
																							fill="none"
																							xmlns="http://www.w3.org/2000/svg"
																							><path
																								d="M3 2C2.44772 2 2 2.44772 2 3V12C2 12.5523 2.44772 13 3 13H12C12.5523 13 13 12.5523 13 12V8.5C13 8.22386 12.7761 8 12.5 8C12.2239 8 12 8.22386 12 8.5V12H3V3L6.5 3C6.77614 3 7 2.77614 7 2.5C7 2.22386 6.77614 2 6.5 2H3ZM12.8536 2.14645C12.9015 2.19439 12.9377 2.24964 12.9621 2.30861C12.9861 2.36669 12.9996 2.4303 13 2.497L13 2.5V2.50049V5.5C13 5.77614 12.7761 6 12.5 6C12.2239 6 12 5.77614 12 5.5V3.70711L6.85355 8.85355C6.65829 9.04882 6.34171 9.04882 6.14645 8.85355C5.95118 8.65829 5.95118 8.34171 6.14645 8.14645L11.2929 3H9.5C9.22386 3 9 2.77614 9 2.5C9 2.22386 9.22386 2 9.5 2H12.4999H12.5C12.5678 2 12.6324 2.01349 12.6914 2.03794C12.7504 2.06234 12.8056 2.09851 12.8536 2.14645Z"
																								fill="currentColor"
																								fill-rule="evenodd"
																								clip-rule="evenodd"
																							></path></svg
																						>
																					</Button>
																					{#if work.orcid == null}
																						<Button style="visibility: hidden"></Button>
																					{:else}
																						<Button
																							variant="ghost"
																							size="sm"
																							class="text text-md"
																							href={work.orcid}
																							target="_blank"
																							rel="noreferrer noopenner"
																							>ORCID record
																							<svg
																								class="margin-left ml-2"
																								width="15"
																								height="15"
																								viewBox="0 0 15 15"
																								fill="none"
																								xmlns="http://www.w3.org/2000/svg"
																								><path
																									d="M3 2C2.44772 2 2 2.44772 2 3V12C2 12.5523 2.44772 13 3 13H12C12.5523 13 13 12.5523 13 12V8.5C13 8.22386 12.7761 8 12.5 8C12.2239 8 12 8.22386 12 8.5V12H3V3L6.5 3C6.77614 3 7 2.77614 7 2.5C7 2.22386 6.77614 2 6.5 2H3ZM12.8536 2.14645C12.9015 2.19439 12.9377 2.24964 12.9621 2.30861C12.9861 2.36669 12.9996 2.4303 13 2.497L13 2.5V2.50049V5.5C13 5.77614 12.7761 6 12.5 6C12.2239 6 12 5.77614 12 5.5V3.70711L6.85355 8.85355C6.65829 9.04882 6.34171 9.04882 6.14645 8.85355C5.95118 8.65829 5.95118 8.34171 6.14645 8.14645L11.2929 3H9.5C9.22386 3 9 2.77614 9 2.5C9 2.22386 9.22386 2 9.5 2H12.4999H12.5C12.5678 2 12.6324 2.01349 12.6914 2.03794C12.7504 2.06234 12.8056 2.09851 12.8536 2.14645Z"
																									fill="currentColor"
																									fill-rule="evenodd"
																									clip-rule="evenodd"
																								></path></svg
																							>
																						</Button>
																					{/if}
																				</div>
																			</div>
																		</Sheet.Header>

																		<div class="grid gap-4 py-4">
																			<div>
																				<hr style="margin-bottom: .5rem" />
																				<span style="font-weight: bold;"> Alternate names: </span>
																				{work.display_name_alternatives} <br />
																				<span style="font-weight: bold;"> Institutions: </span>
																				{#each work.last_known_institutions as last_known}
																					<a
																						href={last_known.id}
																						target="_blank"
																						class="text-blue-500"
																					>
																						{last_known.display_name}
																					</a>

																					<br />
																				{/each}
																				<span style="font-weight: bold;"> Past institutions: </span>
																				{#each work.affiliations as affiliations}
																					<a
																						href={affiliations.institution.id}
																						target="_blank"
																						class="text-blue-500"
																					>
																						{affiliations.institution.display_name}
																					</a>

																					<br />
																				{/each}
																				<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																				<span style="font-weight: bold;"> H-index: </span>
																				{work.summary_stats.h_index} <br />
																				<span style="font-weight: bold;"> I10-index: </span>
																				{work.summary_stats.i10_index} <br />
																				<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																				<span style="font-weight: bold;"> Works count: </span>
																				<a
																					href="https://openalex.org/works?page=1&filter=authorships.author.id:{work.id.substring(
																						'https://openalex.org/'.length
																					)}"
																					target="_blank"
																					rel="noreferrer noopenner"
																					class="text-blue-500"
																				>
																					{work.works_count}
																				</a>

																				<br />
																				<span style="font-weight: bold;"> Citations count: </span>
																				{work.cited_by_count} <br />
																			</div>
																		</div>
																		<!-- <Sheet.Footer>
																			<Sheet.Close asChild let:builder>
																				<Button variant="secondary" builders={[builder]}
																					>Close</Button
																				>
																			</Sheet.Close>
																		</Sheet.Footer> -->
																	</Sheet.Content>
																</Sheet.Root>
															</div>
														</Table.Cell>
													</Table.Row>
												{/each}
											</Table.Body>
										</Table.Root>
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
		<div
			class="container flex-1 items-start md:grid md:grid-cols-[220px_minmax(0,1fr)] md:gap-6 lg:grid-cols-[240px_minmax(0,1fr)] lg:gap-10"
		>
			<main class="relative py-6 lg:gap-10 lg:py-8 xl:grid xl:grid-cols-[1fr_300px]"></main>
		</div>
	</div>
</div>
