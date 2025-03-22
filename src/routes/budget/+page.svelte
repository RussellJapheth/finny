<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import Navigation from '../Navigation.svelte';

	interface Budget {
		id: number;
		categoryId: number;
		amount: number;
		spent: number;
	}

	interface Category {
		id: number;
		name: string;
		icon: string;
	}

	interface Transaction {
		id: number;
		date: string;
		amount: number;
		type: 'income' | 'expense';
		categoryId: number;
		currency: string;
	}

	let budgets: Budget[] = [];
	let categories: Category[] = [];
	let transactions: Transaction[] = [];
	let newBudget = {
		categoryId: 0,
		amount: 0
	};

	onMount(() => {
		const savedBudgets = localStorage.getItem('budgets');
		const savedCategories = localStorage.getItem('categories');
		const savedTransactions = localStorage.getItem('transactions');

		if (savedBudgets) {
			budgets = JSON.parse(savedBudgets);
		}
		if (savedCategories) {
			categories = JSON.parse(savedCategories);
			if (categories.length > 0) {
				newBudget.categoryId = categories[0].id;
			}
		}
		if (savedTransactions) {
			transactions = JSON.parse(savedTransactions);
		}

		calculateSpending();
	});

	function calculateSpending() {
		const now = new Date();
		const currentMonth = now.getMonth();
		const currentYear = now.getFullYear();

		// Filter current month's transactions
		const monthTransactions = transactions.filter((t) => {
			const tDate = new Date(t.date);
			return (
				tDate.getMonth() === currentMonth &&
				tDate.getFullYear() === currentYear &&
				t.type === 'expense'
			);
		});

		// Update spent amounts
		budgets = budgets.map((budget) => {
			const spent = monthTransactions
				.filter((t) => t.categoryId === budget.categoryId)
				.reduce((sum, t) => sum + t.amount, 0);
			return { ...budget, spent };
		});

		localStorage.setItem('budgets', JSON.stringify(budgets));
	}

	function addBudget() {
		if (!newBudget.amount || !newBudget.categoryId) return;

		const budget: Budget = {
			id: Date.now(),
			categoryId: newBudget.categoryId,
			amount: newBudget.amount,
			spent: 0
		};

		budgets = [...budgets, budget];
		localStorage.setItem('budgets', JSON.stringify(budgets));
		calculateSpending();

		newBudget = {
			categoryId: categories[0]?.id || 0,
			amount: 0
		};
	}

	function deleteBudget(id: number) {
		budgets = budgets.filter((b) => b.id !== id);
		localStorage.setItem('budgets', JSON.stringify(budgets));
	}

	function getCategoryName(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.name || 'Uncategorized';
	}

	function getCategoryIcon(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.icon || '📝';
	}

	function formatCurrency(amount: number): string {
		return `₦${amount.toLocaleString('en-NG', { minimumFractionDigits: 2 })}`;
	}

	function calculateProgress(budget: Budget): number {
		return Math.min(Math.round((budget.spent / budget.amount) * 100), 100);
	}
</script>

<div class="flex min-h-screen">
	<Navigation />

	<main class="flex-1 bg-gray-50 p-8">
		<header class="mb-8">
			<h1 class="text-2xl font-bold text-gray-800">Budget</h1>
			<p class="text-gray-600">Manage your monthly spending limits</p>
		</header>

		<!-- Add New Budget -->
		<div class="mb-8 rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Add New Budget</h2>
			<div class="flex gap-4">
				<select
					bind:value={newBudget.categoryId}
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				>
					{#each categories as category}
						<option value={category.id}>{category.name}</option>
					{/each}
				</select>
				<input
					type="number"
					bind:value={newBudget.amount}
					placeholder="Budget amount"
					class="flex-1 rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				/>
				<button
					on:click={addBudget}
					class="rounded-md bg-green-500 px-6 py-2 text-white transition-colors hover:bg-green-600"
				>
					Add Budget
				</button>
			</div>
		</div>

		<!-- Budget List -->
		<div class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-3">
			{#each budgets as budget}
				{@const progress = calculateProgress(budget)}
				<div class="rounded-lg bg-white p-6 shadow-sm">
					<div class="mb-4 flex items-center justify-between">
						<div class="flex items-center gap-3">
							<span class="text-2xl">{getCategoryIcon(budget.categoryId)}</span>
							<h3 class="font-medium text-gray-800">{getCategoryName(budget.categoryId)}</h3>
						</div>
						<button
							on:click={() => deleteBudget(budget.id)}
							class="text-red-500 hover:text-red-600"
						>
							Delete
						</button>
					</div>
					<div class="mb-2">
						<div class="flex justify-between text-sm text-gray-600">
							<span>Spent: {formatCurrency(budget.spent)}</span>
							<span>Budget: {formatCurrency(budget.amount)}</span>
						</div>
					</div>
					<div class="h-2 w-full overflow-hidden rounded-full bg-gray-200">
						<div
							class="h-full rounded-full transition-all duration-300"
							class:bg-green-500={progress < 80}
							class:bg-yellow-500={progress >= 80 && progress < 100}
							class:bg-red-500={progress >= 100}
							style="width: {progress}%"
						/>
					</div>
					<p class="mt-2 text-sm text-gray-600">{progress}% used</p>
				</div>
			{/each}
		</div>
	</main>
</div>
