<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import Navigation from '../Navigation.svelte';

	interface Transaction {
		id: number;
		date: string;
		description: string;
		amount: number;
		type: 'income' | 'expense';
		categoryId: number;
		accountId: number;
		currency: string;
	}

	interface Category {
		id: number;
		name: string;
		icon: string;
	}

	interface Account {
		id: number;
		name: string;
		type: string;
		balance: number;
		currency: string;
	}

	let transactions: Transaction[] = [];
	let categories: Category[] = [];
	let accounts: Account[] = [];
	let newTransaction = {
		date: new Date().toISOString().split('T')[0],
		description: '',
		amount: 0,
		type: 'expense' as const,
		categoryId: 0,
		accountId: 0,
		currency: '₦'
	};

	onMount(() => {
		// Load data from localStorage
		const savedTransactions = localStorage.getItem('transactions');
		const savedCategories = localStorage.getItem('categories');
		const savedAccounts = localStorage.getItem('accounts');

		if (savedTransactions) {
			transactions = JSON.parse(savedTransactions);
		} else {
			// Default transactions
			transactions = [
				{
					id: 1,
					date: '2024-01-15',
					description: 'Salary Payment',
					amount: 250000,
					type: 'income',
					categoryId: 8, // Salary category
					accountId: 1, // Main account
					currency: '₦'
				},
				{
					id: 2,
					date: '2024-01-16',
					description: 'Grocery Shopping',
					amount: 15000,
					type: 'expense',
					categoryId: 1, // Food category
					accountId: 1,
					currency: '₦'
				}
			];
			localStorage.setItem('transactions', JSON.stringify(transactions));
		}

		if (savedCategories) {
			categories = JSON.parse(savedCategories);
		}
		if (savedAccounts) {
			accounts = JSON.parse(savedAccounts);
		}

		// Set default category and account if available
		if (categories.length > 0) {
			newTransaction.categoryId = categories[0].id;
		}
		if (accounts.length > 0) {
			newTransaction.accountId = accounts[0].id;
		}
	});

	function addTransaction() {
		if (!newTransaction.description.trim() || !newTransaction.amount) return;

		const transaction: Transaction = {
			id: Date.now(),
			date: newTransaction.date,
			description: newTransaction.description.trim(),
			amount: newTransaction.amount,
			type: newTransaction.type,
			categoryId: newTransaction.categoryId,
			accountId: newTransaction.accountId,
			currency: newTransaction.currency
		};

		transactions = [transaction, ...transactions];
		localStorage.setItem('transactions', JSON.stringify(transactions));

		// Reset form
		newTransaction = {
			date: new Date().toISOString().split('T')[0],
			description: '',
			amount: 0,
			type: 'expense',
			categoryId: newTransaction.categoryId,
			accountId: newTransaction.accountId,
			currency: '₦'
		};
	}

	function deleteTransaction(id: number) {
		transactions = transactions.filter((t) => t.id !== id);
		localStorage.setItem('transactions', JSON.stringify(transactions));
	}

	function formatCurrency(amount: number, currency: string): string {
		return `${currency}${amount.toLocaleString('en-NG', { minimumFractionDigits: 2 })}`;
	}

	function getCategoryName(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.name || 'Uncategorized';
	}

	function getCategoryIcon(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.icon || '📝';
	}

	function getAccountName(accountId: number): string {
		return accounts.find((a) => a.id === accountId)?.name || 'Unknown Account';
	}
</script>

<div class="flex min-h-screen">
	<Navigation />

	<main class="flex-1 bg-gray-50 p-8">
		<header class="mb-8">
			<h1 class="text-2xl font-bold text-gray-800">Transactions</h1>
			<p class="text-gray-600">Manage your income and expenses</p>
		</header>

		<!-- Add New Transaction -->
		<div class="mb-8 rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Add New Transaction</h2>
			<div class="mb-6 grid gap-4 md:grid-cols-2 lg:grid-cols-3">
				<input
					type="date"
					bind:value={newTransaction.date}
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				/>
				<input
					type="text"
					bind:value={newTransaction.description}
					placeholder="Description"
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				/>
				<input
					type="number"
					bind:value={newTransaction.amount}
					placeholder="Amount"
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				/>
				<select
					bind:value={newTransaction.type}
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				>
					<option value="expense">Expense</option>
					<option value="income">Income</option>
				</select>
				<select
					bind:value={newTransaction.categoryId}
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				>
					{#each categories as category}
						<option value={category.id}>{category.name}</option>
					{/each}
				</select>
				<select
					bind:value={newTransaction.accountId}
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				>
					{#each accounts as account}
						<option value={account.id}>{account.name}</option>
					{/each}
				</select>
			</div>
			<div class="flex justify-end">
				<button
					on:click={addTransaction}
					class="rounded-md bg-green-500 px-6 py-2 text-white transition-colors hover:bg-green-600"
				>
					Add Transaction
				</button>
			</div>
		</div>

		<!-- Transactions List -->
		<div class="rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Recent Transactions</h2>
			<div class="space-y-4">
				{#each transactions as transaction}
					<div
						class="flex items-center justify-between border-b border-gray-100 py-4 last:border-0"
					>
						<div class="flex items-center gap-4">
							<span class="text-2xl">{getCategoryIcon(transaction.categoryId)}</span>
							<div>
								<h4 class="font-medium text-gray-800">{transaction.description}</h4>
								<p class="text-sm text-gray-600">
									{new Date(transaction.date).toLocaleDateString()} • {getCategoryName(
										transaction.categoryId
									)} • {getAccountName(transaction.accountId)}
								</p>
							</div>
						</div>
						<div class="flex items-center gap-4">
							<span
								class={`font-bold ${
									transaction.type === 'income' ? 'text-green-500' : 'text-red-500'
								}`}
							>
								{transaction.type === 'income' ? '+' : '-'}
								{formatCurrency(transaction.amount, transaction.currency)}
							</span>
							<button
								on:click={() => deleteTransaction(transaction.id)}
								class="text-red-500 hover:text-red-600"
							>
								Delete
							</button>
						</div>
					</div>
				{/each}
			</div>
		</div>
	</main>
</div>
