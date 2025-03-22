<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import Navigation from '../Navigation.svelte';

	interface Category {
		id: number;
		name: string;
		icon: string;
	}

	let categories: Category[] = [];
	let newCategory = {
		name: '',
		icon: '🏠' // Set default icon
	};
	let editingCategory: Category | null = null;

	// Add available icons array
	const availableIcons = [
		'🏠',
		'🛒',
		'🍽️',
		'🚗',
		'🏥',
		'📚',
		'🎮',
		'💰',
		'🎵',
		'✈️',
		'👕',
		'💄',
		'🏋️',
		'🎬',
		'🏦',
		'📱',
		'��',
		'⚡',
		'🏢',
		'🎨'
	];

	onMount(() => {
		// Load categories from localStorage
		const savedCategories = localStorage.getItem('categories');
		if (savedCategories) {
			categories = JSON.parse(savedCategories);
		} else {
			// Default categories
			categories = [
				{ id: 1, name: 'Food & Dining', icon: '🍽️' },
				{ id: 2, name: 'Transportation', icon: '🚗' },
				{ id: 3, name: 'Shopping', icon: '🛍️' },
				{ id: 4, name: 'Bills & Utilities', icon: '📄' },
				{ id: 5, name: 'Entertainment', icon: '🎮' },
				{ id: 6, name: 'Healthcare', icon: '🏥' },
				{ id: 7, name: 'Education', icon: '📚' },
				{ id: 8, name: 'Salary', icon: '💰' }
			];
			localStorage.setItem('categories', JSON.stringify(categories));
		}
	});

	function addCategory() {
		if (!newCategory.name.trim()) return;

		const category: Category = {
			id: Date.now(),
			name: newCategory.name.trim(),
			icon: newCategory.icon
		};

		categories = [...categories, category];
		localStorage.setItem('categories', JSON.stringify(categories));

		// Reset form with default icon
		newCategory = {
			name: '',
			icon: '🏠'
		};
	}

	function deleteCategory(id: number) {
		categories = categories.filter((cat) => cat.id !== id);
		localStorage.setItem('categories', JSON.stringify(categories));
	}

	function startEdit(category: Category) {
		editingCategory = { ...category };
	}

	function saveEdit() {
		if (!editingCategory) return;

		categories = categories.map((cat) => (cat.id === editingCategory!.id ? editingCategory! : cat));
		localStorage.setItem('categories', JSON.stringify(categories));
		editingCategory = null;
	}

	function cancelEdit() {
		editingCategory = null;
	}
</script>

<div class="flex min-h-screen">
	<Navigation />

	<main class="flex-1 bg-gray-50 p-8">
		<header class="mb-8">
			<h1 class="text-2xl font-bold text-gray-800">Category Management</h1>
			<p class="text-gray-600">Manage your transaction categories</p>
		</header>

		<!-- Add New Category -->
		<div class="mb-8 rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Add New Category</h2>
			<div class="space-y-4">
				<div class="flex gap-4">
					<select
						bind:value={newCategory.icon}
						class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
					>
						{#each availableIcons as icon}
							<option value={icon}>{icon}</option>
						{/each}
					</select>
					<input
						type="text"
						bind:value={newCategory.name}
						placeholder="Enter category name"
						class="flex-1 rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
					/>
					<button
						on:click={addCategory}
						class="rounded-md bg-green-500 px-6 py-2 text-white transition-colors hover:bg-green-600"
					>
						Add Category
					</button>
				</div>
			</div>
		</div>

		<!-- Categories List -->
		<div class="rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Your Categories</h2>
			<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
				{#each categories as category}
					<div class="rounded-lg border border-gray-200 p-4">
						{#if editingCategory?.id === category.id}
							<!-- Edit Mode -->
							<div class="space-y-4">
								<input
									type="text"
									bind:value={editingCategory.name}
									class="w-full rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
								/>
								<div class="flex gap-2">
									<button
										on:click={saveEdit}
										class="rounded-md bg-green-500 px-4 py-2 text-white transition-colors hover:bg-green-600"
									>
										Save
									</button>
									<button
										on:click={cancelEdit}
										class="rounded-md bg-gray-200 px-4 py-2 text-gray-700 transition-colors hover:bg-gray-300"
									>
										Cancel
									</button>
								</div>
							</div>
						{:else}
							<!-- View Mode -->
							<div class="flex items-center justify-between">
								<div class="flex items-center gap-3">
									<span class="text-2xl">{category.icon}</span>
									<span class="font-medium text-gray-800">{category.name}</span>
								</div>
								<div class="flex gap-2">
									<button
										on:click={() => startEdit(category)}
										class="text-blue-500 hover:text-blue-600"
									>
										Edit
									</button>
									<button
										on:click={() => deleteCategory(category.id)}
										class="text-red-500 hover:text-red-600"
									>
										Delete
									</button>
								</div>
							</div>
						{/if}
					</div>
				{/each}
			</div>
		</div>
	</main>
</div>
