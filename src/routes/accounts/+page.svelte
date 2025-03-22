<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import Navigation from '../Navigation.svelte';

	interface Account {
		id: number;
		name: string;
		type: 'savings' | 'checking' | 'credit' | 'investment';
		balance: number;
		currency: string;
	}

	let accounts: Account[] = [];
	let newAccount = {
		name: '',
		type: 'checking' as const,
		balance: 0,
		currency: '₦'
	};
	let editingAccount: Account | null = null;

	onMount(() => {
		// Load accounts from localStorage
		const savedAccounts = localStorage.getItem('accounts');
		if (savedAccounts) {
			accounts = JSON.parse(savedAccounts);
		} else {
			// Default accounts
			accounts = [
				{ id: 1, name: 'Main Checking', type: 'checking', balance: 25000, currency: '₦' },
				{ id: 2, name: 'Savings', type: 'savings', balance: 50000, currency: '₦' },
				{ id: 3, name: 'Investment Fund', type: 'investment', balance: 100000, currency: '₦' }
			];
			localStorage.setItem('accounts', JSON.stringify(accounts));
		}
	});

	function addAccount() {
		if (!newAccount.name.trim()) return;

		const account: Account = {
			id: Date.now(),
			name: newAccount.name.trim(),
			type: newAccount.type,
			balance: newAccount.balance,
			currency: newAccount.currency
		};

		accounts = [...accounts, account];
		localStorage.setItem('accounts', JSON.stringify(accounts));
		// Reset form
		newAccount = {
			name: '',
			type: 'checking',
			balance: 0,
			currency: '₦'
		};
	}

	function deleteAccount(id: number) {
		accounts = accounts.filter((acc) => acc.id !== id);
		localStorage.setItem('accounts', JSON.stringify(accounts));
	}

	function startEdit(account: Account) {
		editingAccount = { ...account };
	}

	function saveEdit() {
		if (!editingAccount) return;

		accounts = accounts.map((acc) => (acc.id === editingAccount!.id ? editingAccount! : acc));
		localStorage.setItem('accounts', JSON.stringify(accounts));
		editingAccount = null;
	}

	function cancelEdit() {
		editingAccount = null;
	}

	function formatCurrency(amount: number, currency: string): string {
		return `${currency}${amount.toLocaleString('en-NG', { minimumFractionDigits: 2 })}`;
	}
</script>

<div class="flex min-h-screen">
	<Navigation />

	<main class="flex-1 bg-gray-50 p-8">
		<header class="mb-8">
			<h1 class="text-2xl font-bold text-gray-800">Accounts</h1>
			<p class="text-gray-600">Manage your bank accounts and balances</p>
		</header>

		<!-- Add New Account -->
		<div class="mb-8 rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Add New Account</h2>
			<div class="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
				<input
					type="text"
					bind:value={newAccount.name}
					placeholder="Account name"
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				/>
				<select
					bind:value={newAccount.type}
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				>
					<option value="checking">Checking</option>
					<option value="savings">Savings</option>
					<option value="credit">Credit Card</option>
					<option value="investment">Investment</option>
				</select>
				<input
					type="number"
					bind:value={newAccount.balance}
					placeholder="Initial balance"
					class="rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
				/>
				<button
					on:click={addAccount}
					class="rounded-md bg-green-500 px-6 py-2 text-white transition-colors hover:bg-green-600"
				>
					Add Account
				</button>
			</div>
		</div>

		<!-- Accounts List -->
		<div class="rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-lg font-semibold text-gray-800">Your Accounts</h2>
			<div class="grid gap-4 md:grid-cols-2">
				{#each accounts as account}
					<div class="rounded-lg border border-gray-200 p-4">
						{#if editingAccount?.id === account.id}
							<!-- Edit Mode -->
							<div class="space-y-4">
								<input
									type="text"
									bind:value={editingAccount.name}
									class="w-full rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
								/>
								<select
									bind:value={editingAccount.type}
									class="w-full rounded-md border border-gray-300 px-4 py-2 focus:border-green-500 focus:outline-none"
								>
									<option value="checking">Checking</option>
									<option value="savings">Savings</option>
									<option value="credit">Credit Card</option>
									<option value="investment">Investment</option>
								</select>
								<input
									type="number"
									bind:value={editingAccount.balance}
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
							<div class="space-y-2">
								<div class="flex items-center justify-between">
									<div>
										<h3 class="font-medium text-gray-800">{account.name}</h3>
										<p class="text-sm text-gray-600 capitalize">{account.type}</p>
									</div>
									<p class="text-xl font-bold text-gray-800">
										{formatCurrency(account.balance, account.currency)}
									</p>
								</div>
								<div class="flex justify-end gap-2 pt-2">
									<button
										on:click={() => startEdit(account)}
										class="text-blue-500 hover:text-blue-600"
									>
										Edit
									</button>
									<button
										on:click={() => deleteAccount(account.id)}
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
