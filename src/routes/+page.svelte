<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import Navigation from './Navigation.svelte';
	import Chart from 'chart.js/auto';

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
	let totalBalance = 0;
	let monthlyIncome = 0;
	let monthlyExpenses = 0;
	let savingsRate = 0;

	let spendingChartCanvas: HTMLCanvasElement;
	let monthlyChartCanvas: HTMLCanvasElement;

	function initSpendingChart() {
		// Calculate spending by category
		const categoryTotals = transactions
			.filter((t) => t.type === 'expense')
			.reduce(
				(acc, t) => {
					const categoryName = getCategoryName(t.categoryId);
					acc[categoryName] = (acc[categoryName] || 0) + t.amount;
					return acc;
				},
				{} as Record<string, number>
			);

		new Chart(spendingChartCanvas, {
			type: 'doughnut',
			data: {
				labels: Object.keys(categoryTotals),
				datasets: [
					{
						data: Object.values(categoryTotals),
						backgroundColor: [
							'#10B981',
							'#3B82F6',
							'#6366F1',
							'#8B5CF6',
							'#EC4899',
							'#F43F5E',
							'#F97316',
							'#EAB308'
						]
					}
				]
			},
			options: {
				responsive: true,
				plugins: {
					legend: {
						position: 'right'
					}
				}
			}
		});
	}

	function initMonthlyChart() {
		// Get last 6 months of data
		const months = [];
		const incomeData = [];
		const expenseData = [];

		for (let i = 5; i >= 0; i--) {
			const date = new Date();
			date.setMonth(date.getMonth() - i);
			const month = date.toLocaleString('default', { month: 'short' });
			const year = date.getFullYear();
			const monthYear = `${month} ${year}`;

			const monthTransactions = transactions.filter((t) => {
				const tDate = new Date(t.date);
				return tDate.getMonth() === date.getMonth() && tDate.getFullYear() === date.getFullYear();
			});

			const monthlyIncome = monthTransactions
				.filter((t) => t.type === 'income')
				.reduce((sum, t) => sum + t.amount, 0);

			const monthlyExpense = monthTransactions
				.filter((t) => t.type === 'expense')
				.reduce((sum, t) => sum + t.amount, 0);

			months.push(monthYear);
			incomeData.push(monthlyIncome);
			expenseData.push(monthlyExpense);
		}

		new Chart(monthlyChartCanvas, {
			type: 'bar',
			data: {
				labels: months,
				datasets: [
					{
						label: 'Income',
						data: incomeData,
						backgroundColor: '#10B981',
						borderRadius: 4
					},
					{
						label: 'Expenses',
						data: expenseData,
						backgroundColor: '#EF4444',
						borderRadius: 4
					}
				]
			},
			options: {
				responsive: true,
				scales: {
					y: {
						beginAtZero: true,
						ticks: {
							callback: (value) => '₦' + value.toLocaleString()
						}
					}
				},
				plugins: {
					legend: {
						position: 'top'
					}
				}
			}
		});
	}

	onMount(() => {
		// Load data from localStorage
		const savedTransactions = localStorage.getItem('transactions');
		const savedCategories = localStorage.getItem('categories');
		const savedAccounts = localStorage.getItem('accounts');

		if (savedTransactions) {
			transactions = JSON.parse(savedTransactions);
		}
		if (savedCategories) {
			categories = JSON.parse(savedCategories);
		}
		if (savedAccounts) {
			accounts = JSON.parse(savedAccounts);
		}

		// Calculate totals
		calculateTotals();

		// Initialize charts after data is loaded
		initSpendingChart();
		initMonthlyChart();
	});

	function calculateTotals() {
		// Calculate total balance from accounts
		totalBalance = accounts.reduce((sum, account) => sum + account.balance, 0);

		// Get current month's transactions
		const now = new Date();
		const currentMonth = now.getMonth();
		const currentYear = now.getFullYear();

		const currentMonthTransactions = transactions.filter((t) => {
			const transactionDate = new Date(t.date);
			return (
				transactionDate.getMonth() === currentMonth && transactionDate.getFullYear() === currentYear
			);
		});

		// Calculate monthly income and expenses
		monthlyIncome = currentMonthTransactions
			.filter((t) => t.type === 'income')
			.reduce((sum, t) => sum + t.amount, 0);

		monthlyExpenses = currentMonthTransactions
			.filter((t) => t.type === 'expense')
			.reduce((sum, t) => sum + t.amount, 0);

		// Calculate savings rate
		savingsRate =
			monthlyIncome > 0 ? Math.round(((monthlyIncome - monthlyExpenses) / monthlyIncome) * 100) : 0;
	}

	function formatCurrency(amount: number): string {
		return `₦${amount.toLocaleString('en-NG', { minimumFractionDigits: 2 })}`;
	}

	function getCategoryName(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.name || 'Uncategorized';
	}

	function getCategoryIcon(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.icon || '📝';
	}
</script>

<div class="flex min-h-screen">
	<Navigation />

	<!-- Main Content Area -->
	<main class="flex-1 bg-gray-50 p-8">
		<header class="mb-8 flex items-center justify-between">
			<h1 class="text-2xl font-bold text-gray-800">Dashboard</h1>
			<div class="text-gray-600">
				<span>Welcome, User</span>
			</div>
		</header>

		<!-- Financial Overview Cards -->
		<div class="mb-8 grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-4">
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h3 class="text-sm font-medium text-gray-600">Total Balance</h3>
				<p class="mt-2 text-2xl font-bold text-gray-800">{formatCurrency(totalBalance)}</p>
			</div>
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h3 class="text-sm font-medium text-gray-600">Monthly Income</h3>
				<p class="mt-2 text-2xl font-bold text-gray-800">{formatCurrency(monthlyIncome)}</p>
			</div>
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h3 class="text-sm font-medium text-gray-600">Monthly Expenses</h3>
				<p class="mt-2 text-2xl font-bold text-gray-800">{formatCurrency(monthlyExpenses)}</p>
			</div>
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h3 class="text-sm font-medium text-gray-600">Savings Rate</h3>
				<p class="mt-2 text-2xl font-bold text-gray-800">{savingsRate}%</p>
			</div>
		</div>

		<!-- Recent Transactions -->
		<section class="mb-8 rounded-lg bg-white p-6 shadow-sm">
			<h2 class="mb-4 text-xl font-semibold text-gray-800">Recent Transactions</h2>
			<div class="space-y-4">
				{#each transactions.slice(0, 5) as transaction}
					<div class="flex items-center border-b border-gray-100 py-4 last:border-0">
						<div class="mr-4 text-2xl">{getCategoryIcon(transaction.categoryId)}</div>
						<div class="flex-1">
							<h4 class="font-medium text-gray-800">{transaction.description}</h4>
							<p class="text-sm text-gray-600">
								{new Date(transaction.date).toLocaleDateString()} • {getCategoryName(
									transaction.categoryId
								)}
							</p>
						</div>
						<div
							class="font-bold"
							class:text-green-500={transaction.type === 'income'}
							class:text-red-500={transaction.type === 'expense'}
						>
							{transaction.type === 'income' ? '+' : '-'}{formatCurrency(transaction.amount)}
						</div>
					</div>
				{/each}
			</div>
		</section>

		<!-- Charts Section -->
		<section class="grid grid-cols-1 gap-6 lg:grid-cols-2">
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h2 class="mb-4 text-xl font-semibold text-gray-800">Spending by Category</h2>
				<div class="relative mx-auto aspect-square w-full max-w-[400px]">
					<canvas bind:this={spendingChartCanvas}></canvas>
				</div>
			</div>
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h2 class="mb-4 text-xl font-semibold text-gray-800">Monthly Overview</h2>
				<div class="relative h-[300px]">
					<canvas bind:this={monthlyChartCanvas}></canvas>
				</div>
			</div>
		</section>
	</main>
</div>
