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
	let total = 1;
	let totalPages = 1;
	let toPage = 1;

	let autocomplete_works: any[] = [];
	const ror = '04w4pwd42'; // Research Organization Registry ID of President University (https://ror.org/04w4pwd42)

	let autocomplete_authors: any[] = [];
	const id = 'i190243414'; // OpenAlex institution ID of President University (https://openalex.org/institutions/i190243414)

	let list_published: any[] = [];
	let list_statuses: any[] = [];
	let list_types: any[] = [];

	let searchTerm = '';
	let workId = '';
	let workTitle = '';
	let authorId = '';
	let authorName = '';
	let sourceId = '';
	let sourceName = '';
	let pubYear = '';
	let workType = '';
	let statusName = '';

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

	const getWorks = async () => {
		try {
			let api = `https://api.openalex.org/autocomplete/works?filter=institutions.ror:${ror}`;

			if (workTitle.trim() !== '') {
				api += `&q=${encodeURIComponent(workTitle.trim())}`;
			}

			console.log(api);
			const response = await axios.get(api);
			autocomplete_works = response.data.results;
			console.log('getWorks successful', autocomplete_works);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getWorks:', error.message);
		}
	};

	onMount(getWorks);

	const getPublished = async () => {
		try {
			let api = `https://api.openalex.org/works?group_by=publication_year&filter=institutions.ror:${ror}`;
			// let api = `https://api.openalex.org/works?group_by=publication_year&filter=institutions.id:${id}`;

			console.log(api);
			const response = await axios.get(api);
			list_published = response.data.group_by;
			console.log('getPublished successful', list_published);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getPublished:', error.message);
		}
	};

	onMount(getPublished);

	const getTypes = async () => {
		try {
			let api = `https://api.openalex.org/works?group_by=type&filter=institutions.ror:${ror}`;

			console.log(api);
			const response = await axios.get(api);
			list_types = response.data.group_by;
			console.log('getTypes successful', list_types);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getTypes:', error.message);
		}
	};

	onMount(getTypes);

	const getStatuses = async () => {
		try {
			let api = `https://api.openalex.org/works?group_by=oa_status&filter=institutions.ror:${ror}`;

			console.log(api);
			const response = await axios.get(api);
			list_statuses = response.data.group_by;
			console.log('getStatuses successful', list_statuses);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getStatuses:', error.message);
		}
	};

	onMount(getStatuses);

	const fetchData = async () => {
		try {
			let api = `https://api.openalex.org/works?page=${page}&per-page=${pageSize}&sort=publication_date:desc&filter=institutions.ror:${ror}`;

			if (searchTerm.trim() !== '') {
				api += `,default.search:${encodeURIComponent(searchTerm.trim())}`;
			}

			if (authorId.trim() !== '') {
				api += `,authorships.author.id:${encodeURIComponent(authorId.trim())}`;
			}

			if (sourceId.trim() !== '') {
				api += `,primary_location.source.id:${encodeURIComponent(sourceId.trim())}`;
			}

			if (pubYear !== '') {
				api += `,publication_year:${encodeURIComponent(pubYear)}`;
			}

			if (workType !== '') {
				api += `,type:${encodeURIComponent(workType)}`;
			}

			if (statusName !== '') {
				api += `,oa_status:${encodeURIComponent(statusName)}`;
			}

			if (workId !== '') {
				api += `,ids.openalex:${encodeURIComponent(workId)}`;
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

	const autocompleteWork = async (event: any) => {
		workTitle = event.target.value;
		await getWorks();
	};

	function runSearch() {
		page = 1;
		toPage = 1;
		workId = '';
		workTitle = '';
		fetchData();
	}

	function clearSearch() {
		page = 1;
		toPage = 1;
		searchTerm = '';
		workId = '';
		workTitle = '';
		authorId = '';
		authorName = '';
		sourceId = '';
		sourceName = '';
		pubYear = '';
		workType = '';
		statusName = '';
		fetchData();
	}

	function clearWork() {
		page = 1;
		toPage = 1;
		searchTerm = '';
		workId = '';
		workTitle = '';
		fetchData();
	}

	function clearAuthor() {
		page = 1;
		toPage = 1;
		authorId = '';
		authorName = '';
		fetchData();
	}

	function clearSource() {
		page = 1;
		toPage = 1;
		sourceId = '';
		sourceName = '';
		fetchData();
	}

	function clearPublished() {
		page = 1;
		toPage = 1;
		pubYear = '';
		fetchData();
	}

	function clearType() {
		page = 1;
		toPage = 1;
		workType = '';
		fetchData();
	}

	function clearStatus() {
		page = 1;
		toPage = 1;
		statusName = '';
		fetchData();
	}

	function copyAuthorId(id: string) {
		navigator.clipboard
			.writeText(id)
			.then(() => {
				authorId = id;
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

	function copyWorkId(id: string) {
		navigator.clipboard
			.writeText(id)
			.then(() => {
				searchTerm = '';
				workId = id;
				page = 1;
				toPage = 1;
				fetchData();
			})
			.catch((error) => {
				console.error('Failed to copy:', error);
			});
	}

	function copyWorkTitle(name: string) {
		navigator.clipboard;
		workTitle = name;
	}

	function copySourceId(id: string) {
		navigator.clipboard
			.writeText(id)
			.then(() => {
				sourceId = id;
				page = 1;
				toPage = 1;
				fetchData();
			})
			.catch((error) => {
				console.error('Failed to copy:', error);
			});
	}

	function copySourceName(name: string) {
		navigator.clipboard;
		sourceName = name;
	}

	function copyPubYear(year: string) {
		navigator.clipboard
			.writeText(year)
			.then(() => {
				pubYear = year;
				page = 1;
				toPage = 1;
				fetchData();
			})
			.catch((error: any) => {
				console.error('Failed to copy:', error);
			});
	}

	function copyWorkType(type: string) {
		navigator.clipboard
			.writeText(type)
			.then(() => {
				workType = type;
				page = 1;
				toPage = 1;
				fetchData();
			})
			.catch((error: any) => {
				console.error('Failed to copy:', error);
			});
	}

	function copyStatus(name: string) {
		navigator.clipboard
			.writeText(name)
			.then(() => {
				statusName = name;
				page = 1;
				toPage = 1;
				fetchData();
			})
			.catch((error: any) => {
				console.error('Failed to copy:', error);
			});
	}
</script>

<div class="flex-1">
	<div class="border-b">
		<div class="mx-auto w-full min-w-0 max-w-screen-2xl" style="padding: 2rem;">
			<div class="space-y-2">
				<h1 class="scroll-m-20 text-4xl font-bold tracking-tight">Works</h1>
				<p class="text-balance text-lg text-muted-foreground">
					Journal articles, books, datasets, and theses.
				</p>
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
																		d="M4.5 1C4.77614 1 5 1.22386 5 1.5V2H10V1.5C10 1.22386 10.2239 1 10.5 1C10.7761 1 11 1.22386 11 1.5V2H12.5C13.3284 2 14 2.67157 14 3.5V12.5C14 13.3284 13.3284 14 12.5 14H2.5C1.67157 14 1 13.3284 1 12.5V3.5C1 2.67157 1.67157 2 2.5 2H4V1.5C4 1.22386 4.22386 1 4.5 1ZM10 3V3.5C10 3.77614 10.2239 4 10.5 4C10.7761 4 11 3.77614 11 3.5V3H12.5C12.7761 3 13 3.22386 13 3.5V5H2V3.5C2 3.22386 2.22386 3 2.5 3H4V3.5C4 3.77614 4.22386 4 4.5 4C4.77614 4 5 3.77614 5 3.5V3H10ZM2 6V12.5C2 12.7761 2.22386 13 2.5 13H12.5C12.7761 13 13 12.7761 13 12.5V6H2ZM7 7.5C7 7.22386 7.22386 7 7.5 7C7.77614 7 8 7.22386 8 7.5C8 7.77614 7.77614 8 7.5 8C7.22386 8 7 7.77614 7 7.5ZM9.5 7C9.22386 7 9 7.22386 9 7.5C9 7.77614 9.22386 8 9.5 8C9.77614 8 10 7.77614 10 7.5C10 7.22386 9.77614 7 9.5 7ZM11 7.5C11 7.22386 11.2239 7 11.5 7C11.7761 7 12 7.22386 12 7.5C12 7.77614 11.7761 8 11.5 8C11.2239 8 11 7.77614 11 7.5ZM11.5 9C11.2239 9 11 9.22386 11 9.5C11 9.77614 11.2239 10 11.5 10C11.7761 10 12 9.77614 12 9.5C12 9.22386 11.7761 9 11.5 9ZM9 9.5C9 9.22386 9.22386 9 9.5 9C9.77614 9 10 9.22386 10 9.5C10 9.77614 9.77614 10 9.5 10C9.22386 10 9 9.77614 9 9.5ZM7.5 9C7.22386 9 7 9.22386 7 9.5C7 9.77614 7.22386 10 7.5 10C7.77614 10 8 9.77614 8 9.5C8 9.22386 7.77614 9 7.5 9ZM5 9.5C5 9.22386 5.22386 9 5.5 9C5.77614 9 6 9.22386 6 9.5C6 9.77614 5.77614 10 5.5 10C5.22386 10 5 9.77614 5 9.5ZM3.5 9C3.22386 9 3 9.22386 3 9.5C3 9.77614 3.22386 10 3.5 10C3.77614 10 4 9.77614 4 9.5C4 9.22386 3.77614 9 3.5 9ZM3 11.5C3 11.2239 3.22386 11 3.5 11C3.77614 11 4 11.2239 4 11.5C4 11.7761 3.77614 12 3.5 12C3.22386 12 3 11.7761 3 11.5ZM5.5 11C5.22386 11 5 11.2239 5 11.5C5 11.7761 5.22386 12 5.5 12C5.77614 12 6 11.7761 6 11.5C6 11.2239 5.77614 11 5.5 11ZM7 11.5C7 11.2239 7.22386 11 7.5 11C7.77614 11 8 11.2239 8 11.5C8 11.7761 7.77614 12 7.5 12C7.22386 12 7 11.7761 7 11.5ZM9.5 11C9.22386 11 9 11.2239 9 11.5C9 11.7761 9.22386 12 9.5 12C9.77614 12 10 11.7761 10 11.5C10 11.2239 9.77614 11 9.5 11Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
															</Button>
														</Dialog.Trigger>
														<Dialog.Content class="padding p-0">
															<Command.Root>
																<Command.Input placeholder="Type publication year" />
																{#if pubYear !== ''}
																	<Command.Group heading="Actions">
																		<Command.Item>
																			<button
																				class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																				on:click={clearPublished}
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
																	<Command.Group heading="Publication Year">
																		<Command.Empty>No results found.</Command.Empty>
																		{#each list_published as published}
																			{#if published.count < 1}
																				<div></div>
																			{:else}
																				<Command.Item>
																					<button
																						class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																						on:click={() => copyPubYear(published.key_display_name)}
																					>
																						<t
																							class="text overflow-auto whitespace-normal text-left"
																						>
																							{published.key_display_name}
																							<!-- ({published.count}) -->
																						</t>
																					</button>
																				</Command.Item>
																			{/if}
																		{/each}
																	</Command.Group>
																	<Command.Separator />
																</Command.List>
																{#if pubYear !== ''}
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
																			Filter active: {pubYear}
																		</p>
																	</div>
																{:else}
																	<div></div>
																{/if}
															</Command.Root>
														</Dialog.Content>
													</div>
												</Dialog.Root>

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
																		d="M3.5 2C3.22386 2 3 2.22386 3 2.5V12.5C3 12.7761 3.22386 13 3.5 13H11.5C11.7761 13 12 12.7761 12 12.5V6H8.5C8.22386 6 8 5.77614 8 5.5V2H3.5ZM9 2.70711L11.2929 5H9V2.70711ZM2 2.5C2 1.67157 2.67157 1 3.5 1H8.5C8.63261 1 8.75979 1.05268 8.85355 1.14645L12.8536 5.14645C12.9473 5.24021 13 5.36739 13 5.5V12.5C13 13.3284 12.3284 14 11.5 14H3.5C2.67157 14 2 13.3284 2 12.5V2.5Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
															</Button>
														</Dialog.Trigger>
														<Dialog.Content class="padding p-0">
															<Command.Root>
																<Command.Input placeholder="Search work type" />
																<div
																	class="
															padding relative flex cursor-default select-none items-center
															rounded-sm p-2 text-sm outline-none hover:text-blue-500
															aria-selected:bg-accent aria-selected:text-accent-foreground
															data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm text-orange-700 dark:text-orange-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Work types count assumes other filters are inactive
																	</p>
																</div>
																<Command.Separator />

																{#if workType !== ''}
																	<Command.Group heading="Actions">
																		<Command.Item>
																			<button
																				class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																				on:click={clearType}
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
																	<Command.Group heading="Types">
																		<Command.Empty>No results found.</Command.Empty>
																		{#each list_types as type}
																			{#if type.count < 1}
																				<div></div>
																			{:else}
																				<Command.Item>
																					<button
																						class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																						on:click={() => copyWorkType(type.key_display_name)}
																					>
																						<t
																							class="text overflow-auto whitespace-normal text-left"
																						>
																							{type.key_display_name} ({type.count})
																						</t>
																					</button>
																				</Command.Item>
																			{/if}
																		{/each}
																	</Command.Group>
																	<Command.Separator />
																</Command.List>
																{#if workType !== ''}
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
																			Filter active: {workType}
																		</p>
																	</div>
																{:else}
																	<div></div>
																{/if}
															</Command.Root>
														</Dialog.Content>
													</div>
												</Dialog.Root>

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
																		d="M7.5 11C4.80285 11 2.52952 9.62184 1.09622 7.50001C2.52952 5.37816 4.80285 4 7.5 4C10.1971 4 12.4705 5.37816 13.9038 7.50001C12.4705 9.62183 10.1971 11 7.5 11ZM7.5 3C4.30786 3 1.65639 4.70638 0.0760002 7.23501C-0.0253338 7.39715 -0.0253334 7.60288 0.0760014 7.76501C1.65639 10.2936 4.30786 12 7.5 12C10.6921 12 13.3436 10.2936 14.924 7.76501C15.0253 7.60288 15.0253 7.39715 14.924 7.23501C13.3436 4.70638 10.6921 3 7.5 3ZM7.5 9.5C8.60457 9.5 9.5 8.60457 9.5 7.5C9.5 6.39543 8.60457 5.5 7.5 5.5C6.39543 5.5 5.5 6.39543 5.5 7.5C5.5 8.60457 6.39543 9.5 7.5 9.5Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
															</Button>
														</Dialog.Trigger>
														<Dialog.Content class="padding p-0">
															<Command.Root>
																<Command.Input placeholder="Type access status" />
																<div
																	class="
															padding relative flex cursor-default select-none items-center
															rounded-sm p-2 text-sm outline-none hover:text-blue-500
															aria-selected:bg-accent aria-selected:text-accent-foreground
															data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm text-orange-700 dark:text-orange-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Access status count assumes other filters are inactive
																	</p>
																</div>
																<Command.Separator />
																{#if statusName !== ''}
																	<Command.Group heading="Actions">
																		<Command.Item>
																			<button
																				class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																				on:click={clearStatus}
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
																	<Command.Group heading="Access Status">
																		<Command.Empty>No results found.</Command.Empty>
																		{#each list_statuses as status}
																			{#if status.count < 1}
																				<div></div>
																			{:else}
																				<Command.Item>
																					<button
																						class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																						on:click={() => copyStatus(status.key_display_name)}
																					>
																						<t
																							class="text overflow-auto whitespace-normal text-left"
																						>
																							{status.key_display_name} ({status.count})
																						</t>
																					</button>
																				</Command.Item>
																			{/if}
																		{/each}
																	</Command.Group>
																	<Command.Separator />
																</Command.List>
																{#if statusName !== ''}
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
																			Filter active: {statusName}
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
															class="flex h-9 w-full max-w-sm rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 only:focus-visible:outline-none disabled:cursor-not-allowed disabled:opacity-50"
															type="text"
															bind:value={searchTerm}
															placeholder="Title, abstract, fulltext"
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
												{#if workId !== '' || searchTerm == ''}
													<Button size="icon" style="visibility: hidden"></Button>
												{:else}
													<Button variant="ghost" size="icon" on:click={clearWork}>
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

												<Dialog.Root>
													<div class="w-full flex-1 md:w-auto md:flex-none">
														<Dialog.Trigger>
															{#if workId !== ''}
																<Button size="sm" variant="ghost">
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
																	>Work filter active
																</Button>
															{:else}
																<Button size="sm" variant="outline">
																	<svg
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M3 2.5C3 2.22386 3.22386 2 3.5 2H9.08579C9.21839 2 9.34557 2.05268 9.43934 2.14645L11.8536 4.56066C11.9473 4.65443 12 4.78161 12 4.91421V12.5C12 12.7761 11.7761 13 11.5 13H3.5C3.22386 13 3 12.7761 3 12.5V2.5ZM3.5 1C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H11.5C12.3284 14 13 13.3284 13 12.5V4.91421C13 4.51639 12.842 4.13486 12.5607 3.85355L10.1464 1.43934C9.86514 1.15804 9.48361 1 9.08579 1H3.5ZM4.5 4C4.22386 4 4 4.22386 4 4.5C4 4.77614 4.22386 5 4.5 5H7.5C7.77614 5 8 4.77614 8 4.5C8 4.22386 7.77614 4 7.5 4H4.5ZM4.5 7C4.22386 7 4 7.22386 4 7.5C4 7.77614 4.22386 8 4.5 8H10.5C10.7761 8 11 7.77614 11 7.5C11 7.22386 10.7761 7 10.5 7H4.5ZM4.5 10C4.22386 10 4 10.2239 4 10.5C4 10.7761 4.22386 11 4.5 11H10.5C10.7761 11 11 10.7761 11 10.5C11 10.2239 10.7761 10 10.5 10H4.5Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																</Button>
															{/if}
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
																		bind:value={workTitle}
																		on:input={autocompleteWork}
																		placeholder="Type work title"
																	/>
																</div>
																{#if workId !== ''}
																	<Command.Group heading="Actions">
																		<Command.Item>
																			<button
																				class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																				on:click={clearWork}
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
																	<Command.Group heading="Works">
																		<Command.Empty>No results found.</Command.Empty>
																		{#each autocomplete_works as work}
																			<Command.Item>
																				<button
																					class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() =>
																						copyWorkId(
																							work.id.substring('https://openalex.org/'.length)
																						)}
																					on:click={() => copyWorkTitle(work.display_name)}
																				>
																					<t class="text overflow-auto whitespace-normal text-left">
																						{work.display_name}
																					</t>
																				</button>
																			</Command.Item>
																		{/each}
																	</Command.Group>
																	<Command.Separator />
																</Command.List>
																{#if workId !== ''}
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
																			{#if total <= 0}
																				Filter active: no results found
																			{:else}
																				{#each items as work}
																					Filter active: {work.id.substring('https://'.length)}
																				{/each}
																			{/if}
																		</p>
																	</div>
																{:else}
																	<div></div>
																{/if}
															</Command.Root>
														</Dialog.Content>
													</div>
												</Dialog.Root>

												<t class="text-sm font-medium text-muted-foreground">
													<span class="font-bold">
														{total}
													</span>
													{#if total > 1 || total == 0}
														results
													{:else}
														result
													{/if}
													{#if authorId !== ''}
														by
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
												<t class="text-sm font-medium text-muted-foreground">
													{#if sourceId !== ''}
														from
														<Button
															variant="ghost"
															size="sm"
															class="text text-md"
															on:click={clearSource}
														>
															{sourceName}
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
												<t class="text-sm font-medium text-muted-foreground">
													{#if pubYear !== ''}
														published in
														<Button
															variant="ghost"
															size="sm"
															class="text text-md"
															on:click={clearPublished}
														>
															{pubYear}
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
													<!-- {#if items.length < pageSize} -->
													{#if page == totalPages}
														<div style="visibility: hidden;">
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
														{#if workType == ''}
															Works
														{:else}
															<Button
																variant="ghost"
																size="sm"
																class="text text-md"
																on:click={clearType}
															>
																{workType}
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
														{/if}
													</Table.Head>
													<Table.Head class="text-right">
														{#if statusName == ''}
															Status
														{:else}
															<Button
																variant="ghost"
																size="sm"
																class="text text-md"
																style="text-transform: capitalize"
																on:click={clearStatus}
															>
																{statusName}
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
														{/if}
													</Table.Head>
												</Table.Row>
											</Table.Header>
											<Table.Body>
												{#each items as work}
													<Table.Row>
														<Table.Cell class="font-medium">
															<a href={work.doi} target="_blank" rel="noreferrer noopener">
																{work.title}
															</a>
															<div style="display: flex; " class="text-muted-foreground">
																<div>
																	<!-- class="text-md" -->
																	<Button
																		variant="ghost"
																		size="sm"
																		class="text text-md"
																		on:click={() => copyPubYear(work.publication_year)}
																		>{work.publication_date + ''}</Button
																	>
																	{#each work.authorships as author}
																		<!-- class="text-md" -->
																		<Button
																			variant="ghost"
																			size="sm"
																			class="text text-md"
																			on:click={() =>
																				copyAuthorId(
																					author.author.id.substring('https://openalex.org/'.length)
																				)}
																			on:click={() => copyAuthorName(author.author.display_name)}
																			>{author.author.display_name}</Button
																		>
																	{/each}
																	{#if work.primary_location != null}
																		{#if work.primary_location.source != null}
																			<!-- class="text-md" -->
																			<Button
																				variant="ghost"
																				size="sm"
																				class="text text-md"
																				style="font-style: italic;"
																				on:click={() =>
																					copySourceId(
																						work.primary_location.source.id.substring(
																							'https://openalex.org/'.length
																						)
																					)}
																				on:click={() =>
																					copySourceName(work.primary_location.source.display_name)}
																				>{work.primary_location.source.display_name}</Button
																			>
																		{:else}
																			<div></div>
																		{/if}
																	{:else}
																		<div></div>
																	{/if}
																</div>
															</div>
															<div style="display: flex; ">
																{#if work.cited_by_count > 0}
																	<div>
																		<!-- class="text-md font-semibold" -->
																		<Button
																			class="font-semibold"
																			variant="ghost"
																			size="sm"
																			href="https://openalex.org/works?page=1&filter=cites:{work.id.substring(
																				'https://openalex.org/'.length
																			)}"
																			target="_blank"
																			rel="noreferrer noopener"
																		>
																			Cited by {work.cited_by_count}
																		</Button>
																	</div>
																{:else}
																	<div></div>
																{/if}
																{#if work.primary_location != null}
																	{#if work.primary_location.pdf_url != null}
																		<div>
																			<!-- class="text-md font-semibold" -->
																			<Button
																				class="font-semibold"
																				variant="ghost"
																				size="sm"
																				href={work.primary_location.pdf_url}
																				target="_blank"
																				rel="noreferrer noopener"
																			>
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M3 2.5C3 2.22386 3.22386 2 3.5 2H9.08579C9.21839 2 9.34557 2.05268 9.43934 2.14645L11.8536 4.56066C11.9473 4.65443 12 4.78161 12 4.91421V12.5C12 12.7761 11.7761 13 11.5 13H3.5C3.22386 13 3 12.7761 3 12.5V2.5ZM3.5 1C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H11.5C12.3284 14 13 13.3284 13 12.5V4.91421C13 4.51639 12.842 4.13486 12.5607 3.85355L10.1464 1.43934C9.86514 1.15804 9.48361 1 9.08579 1H3.5ZM4.5 4C4.22386 4 4 4.22386 4 4.5C4 4.77614 4.22386 5 4.5 5H7.5C7.77614 5 8 4.77614 8 4.5C8 4.22386 7.77614 4 7.5 4H4.5ZM4.5 7C4.22386 7 4 7.22386 4 7.5C4 7.77614 4.22386 8 4.5 8H10.5C10.7761 8 11 7.77614 11 7.5C11 7.22386 10.7761 7 10.5 7H4.5ZM4.5 10C4.22386 10 4 10.2239 4 10.5C4 10.7761 4.22386 11 4.5 11H10.5C10.7761 11 11 10.7761 11 10.5C11 10.2239 10.7761 10 10.5 10H4.5Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				PDF
																			</Button>
																		</div>
																	{:else}
																		<div></div>
																	{/if}
																{:else}
																	<div></div>
																{/if}
															</div>
														</Table.Cell>
														<Table.Cell class="text-right">
															<div>
																<Sheet.Root>
																	<Sheet.Trigger asChild let:builder>
																		<Button
																			builders={[builder]}
																			variant="ghost"
																			style="text-transform: uppercase"
																		>
																			<!-- class="text-md" -->
																			{#if work.open_access.oa_status == 'closed'}
																				<t class="text-muted-foreground">
																					{work.open_access.oa_status}
																				</t>
																			{:else if work.open_access.oa_status == 'diamond'}
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
																				{work.open_access.oa_status}
																			{:else}
																				{work.open_access.oa_status}
																			{/if}
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
																	</Sheet.Trigger>
																	<Sheet.Content
																		side="right"
																		class="h-screen max-h-max overflow-y-auto"
																	>
																		<Sheet.Header>
																			<Sheet.Title>{work.title}</Sheet.Title>
																			<Sheet.Description>
																				<Button
																					variant="ghost"
																					size="sm"
																					class="text text-md"
																					href="https://openalex.org/works/{work.id.substring(
																						'https://openalex.org/'.length
																					)}"
																					target="_blank"
																					rel="noreferrer noopenner"
																				>
																					{work.type}
																				</Button>
																			</Sheet.Description>
																			<div style="display:inline-flex; ">
																				<div>
																					{#if work.open_access.oa_status == 'closed'}
																						<Button
																							variant="ghost"
																							size="sm"
																							class="text text-md"
																							href={work.doi}
																							target="_blank"
																							rel="noreferrer noopener"
																						>
																							HTML
																							<svg
																								class="margin-left ml-2"
																								width="15"
																								height="15"
																								viewBox="0 0 15 15"
																								fill="none"
																								xmlns="http://www.w3.org/2000/svg"
																								><path
																									d="M5 4.63601C5 3.76031 5.24219 3.1054 5.64323 2.67357C6.03934 2.24705 6.64582 1.9783 7.5014 1.9783C8.35745 1.9783 8.96306 2.24652 9.35823 2.67208C9.75838 3.10299 10 3.75708 10 4.63325V5.99999H5V4.63601ZM4 5.99999V4.63601C4 3.58148 4.29339 2.65754 4.91049 1.99307C5.53252 1.32329 6.42675 0.978302 7.5014 0.978302C8.57583 0.978302 9.46952 1.32233 10.091 1.99162C10.7076 2.65557 11 3.57896 11 4.63325V5.99999H12C12.5523 5.99999 13 6.44771 13 6.99999V13C13 13.5523 12.5523 14 12 14H3C2.44772 14 2 13.5523 2 13V6.99999C2 6.44771 2.44772 5.99999 3 5.99999H4ZM3 6.99999H12V13H3V6.99999Z"
																									fill="currentColor"
																									fill-rule="evenodd"
																									clip-rule="evenodd"
																								></path></svg
																							>
																						</Button>
																					{:else}
																						<Button
																							variant="default"
																							size="sm"
																							class="text text-md"
																							href={work.doi}
																							target="_blank"
																							rel="noreferrer noopener"
																						>
																							HTML
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
																				{#if work.primary_location != null}
																					{#if work.primary_location.pdf_url != null}
																						<div>
																							<Button
																								variant="ghost"
																								size="sm"
																								class="text text-md"
																								href={work.primary_location.pdf_url}
																								target="_blank"
																								rel="noreferrer noopener"
																							>
																								<svg
																									class="margin-right mr-2"
																									width="15"
																									height="15"
																									viewBox="0 0 15 15"
																									fill="none"
																									xmlns="http://www.w3.org/2000/svg"
																									><path
																										d="M3 2.5C3 2.22386 3.22386 2 3.5 2H9.08579C9.21839 2 9.34557 2.05268 9.43934 2.14645L11.8536 4.56066C11.9473 4.65443 12 4.78161 12 4.91421V12.5C12 12.7761 11.7761 13 11.5 13H3.5C3.22386 13 3 12.7761 3 12.5V2.5ZM3.5 1C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H11.5C12.3284 14 13 13.3284 13 12.5V4.91421C13 4.51639 12.842 4.13486 12.5607 3.85355L10.1464 1.43934C9.86514 1.15804 9.48361 1 9.08579 1H3.5ZM4.5 4C4.22386 4 4 4.22386 4 4.5C4 4.77614 4.22386 5 4.5 5H7.5C7.77614 5 8 4.77614 8 4.5C8 4.22386 7.77614 4 7.5 4H4.5ZM4.5 7C4.22386 7 4 7.22386 4 7.5C4 7.77614 4.22386 8 4.5 8H10.5C10.7761 8 11 7.77614 11 7.5C11 7.22386 10.7761 7 10.5 7H4.5ZM4.5 10C4.22386 10 4 10.2239 4 10.5C4 10.7761 4.22386 11 4.5 11H10.5C10.7761 11 11 10.7761 11 10.5C11 10.2239 10.7761 10 10.5 10H4.5Z"
																										fill="currentColor"
																										fill-rule="evenodd"
																										clip-rule="evenodd"
																									></path></svg
																								>
																								PDF
																							</Button>
																						</div>
																					{:else}
																						<div></div>
																					{/if}
																				{:else}
																					<div></div>
																				{/if}
																			</div>
																		</Sheet.Header>
																		<div class="grid gap-4 py-4">
																			<div>
																				<hr style="margin-bottom: .5rem" />
																				<span style="font-weight: bold;"> Published: </span>
																				{work.publication_date}
																				<br />
																				{#if work.primary_location != null}
																					{#if work.primary_location.source != null}
																						<span style="font-weight: bold;"> Source: </span>
																						<a
																							href={work.primary_location.source.id}
																							target="_blank"
																							rel="noreferrer noopener"
																							class="text-blue-500"
																						>
																							{work.primary_location.source.display_name}
																						</a> <br />
																					{:else}
																						<div></div>
																					{/if}
																				{:else}
																					<div></div>
																				{/if}
																				<span style="font-weight: bold;"> Authors: </span>
																				{#each work.authorships as author}
																					<a
																						href={author.author.id}
																						target="_blank"
																						rel="noreferrer noopener"
																						class="space-around -flex items-center gap-1 rounded-md border border-transparent px-2.5 py-0.5 text-blue-500 transition-colors hover:bg-secondary/80 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
																					>
																						{author.author.display_name + ' '}
																					</a>
																				{/each} <br />
																				<span style="font-weight: bold;"> Institutions: </span>
																				{#each work.authorships as author}
																					{#if author.institutions[0] != undefined}
																						<a
																							href={author.institutions[0].id}
																							target="_blank"
																							rel="noreferrer noopener"
																							class="text-blue-500"
																						>
																							{author.institutions[0].display_name}
																						</a>
																						<br />
																					{:else}
																						<div></div>
																					{/if}
																				{/each}
																				<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																				{#if work.referenced_works_count > 0}
																					<span style="font-weight: bold;"> Cites: </span>
																					<a
																						href="https://openalex.org/works?page=1&filter=cited_by:{work.id.substring(
																							'https://openalex.org/'.length
																						)}"
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.referenced_works_count}
																					</a> <br />
																				{:else}
																					<span></span>
																				{/if}
																				{#if work.cited_by_count > 0}
																					<span style="font-weight: bold;"> Cited by: </span>
																					<a
																						href="https://openalex.org/works?page=1&filter=cites:{work.id.substring(
																							'https://openalex.org/'.length
																						)}"
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.cited_by_count}
																					</a> <br />
																				{:else}
																					<span></span>
																				{/if}
																				{#if work.related_works.length > 0}
																					<span style="font-weight: bold;"> Related to: </span>
																					<a
																						href="https://openalex.org/works?page=1&filter=related_to:{work.id.substring(
																							'https://openalex.org/'.length
																						)}"
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.related_works.length}
																					</a> <br />
																				{:else}
																					<span></span>
																				{/if}
																				<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																				{#if work.primary_topic == null}
																					<div></div>
																				{:else}
																					<span style="font-weight: bold;"> Topic: </span>
																					<a
																						href={work.primary_topic.id}
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.primary_topic.display_name}
																					</a> <br />
																					<span style="font-weight: bold;"> Subfield: </span>
																					<a
																						href={work.primary_topic.subfield.id}
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.primary_topic.subfield.display_name}
																					</a> <br />
																					<span style="font-weight: bold;"> Field: </span>
																					<a
																						href={work.primary_topic.field.id}
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.primary_topic.field.display_name}
																					</a> <br />
																					<span style="font-weight: bold;"> Domain: </span>
																					<a
																						href={work.primary_topic.domain.id}
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.primary_topic.domain.display_name}
																					</a> <br />
																				{/if}
																				{#if work.sustainable_development_goals == undefined || work.sustainable_development_goals.length == 0}
																					<div></div>
																				{:else}
																					<span style="font-weight: bold;">
																						Sustainable Development Goal:
																					</span>
																					<a
																						href={work.sustainable_development_goals[0].id}
																						target="_blank"
																						rel="noreferrer noopener"
																						class="text-blue-500"
																					>
																						{work.sustainable_development_goals[0].display_name}
																					</a> <br />
																				{/if}
																				<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																				<span style="font-weight: bold;">
																					Open Access Status:
																				</span>
																				<Sheet.Close asChild let:builder>
																					<Button
																						builders={[builder]}
																						variant="ghost"
																						size="sm"
																						class="text-base font-normal text-blue-500 transition-colors hover:bg-secondary/80 hover:text-blue-500"
																						style="text-transform: capitalize"
																						on:click={() => copyStatus(work.open_access.oa_status)}
																						>{work.open_access.oa_status}
																						<svg
																							class="margin-left ml-2"
																							width="15"
																							height="15"
																							viewBox="0 0 15 15"
																							fill="none"
																							xmlns="http://www.w3.org/2000/svg"
																							><path
																								d="M8 2.75C8 2.47386 7.77614 2.25 7.5 2.25C7.22386 2.25 7 2.47386 7 2.75V7H2.75C2.47386 7 2.25 7.22386 2.25 7.5C2.25 7.77614 2.47386 8 2.75 8H7V12.25C7 12.5261 7.22386 12.75 7.5 12.75C7.77614 12.75 8 12.5261 8 12.25V8H12.25C12.5261 8 12.75 7.77614 12.75 7.5C12.75 7.22386 12.5261 7 12.25 7H8V2.75Z"
																								fill="currentColor"
																								fill-rule="evenodd"
																								clip-rule="evenodd"
																							></path></svg
																						>
																					</Button>
																				</Sheet.Close>
																				<br />
																			</div>
																			<Sheet.Footer>
																				<Sheet.Close asChild let:builder>
																					<Button
																						builders={[builder]}
																						variant="secondary"
																						on:click={() => copyWorkType(work.type)}
																						>Filter by type - {work.type}
																					</Button>
																				</Sheet.Close>
																			</Sheet.Footer>
																		</div></Sheet.Content
																	>
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
