<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/state';
	import Navigation from '../Navigation.svelte';
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

	let transactions: Transaction[] = [];
	let categories: Category[] = [];
	let selectedMonth = new Date().toISOString().slice(0, 7); // YYYY-MM format
	let reportChart: Chart | null = null;
	let trendChart: Chart | null = null;
	let reportCanvas: HTMLCanvasElement;
	let trendCanvas: HTMLCanvasElement;

	onMount(() => {
		const savedTransactions = localStorage.getItem('transactions');
		const savedCategories = localStorage.getItem('categories');

		if (savedTransactions) transactions = JSON.parse(savedTransactions);
		if (savedCategories) categories = JSON.parse(savedCategories);

		generateCharts();
	});

	function generateCharts() {
		if (!reportCanvas || !trendCanvas) return;

		// Cleanup existing charts
		if (reportChart) reportChart.destroy();
		if (trendChart) trendChart.destroy();

		const [year, month] = selectedMonth.split('-').map(Number);

		// Get transactions for selected month
		const monthTransactions = transactions.filter((t) => {
			const tDate = new Date(t.date);
			return tDate.getMonth() === month - 1 && tDate.getFullYear() === year;
		});

		// Category breakdown chart
		const categoryData = categories
			.map((category) => {
				const total = monthTransactions
					.filter((t) => t.categoryId === category.id && t.type === 'expense')
					.reduce((sum, t) => sum + t.amount, 0);
				return {
					category: category.name,
					total
				};
			})
			.filter((item) => item.total > 0);

		reportChart = new Chart(reportCanvas, {
			type: 'pie',
			data: {
				labels: categoryData.map((d) => d.category),
				datasets: [
					{
						data: categoryData.map((d) => d.total),
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

		// Daily spending trend
		const dailySpending = monthTransactions
			.filter((t) => t.type === 'expense')
			.reduce(
				(acc, t) => {
					const day = new Date(t.date).getDate();
					acc[day] = (acc[day] || 0) + t.amount;
					return acc;
				},
				{} as Record<number, number>
			);

		const days = Array.from({ length: 31 }, (_, i) => i + 1);

		trendChart = new Chart(trendCanvas, {
			type: 'line',
			data: {
				labels: days,
				datasets: [
					{
						label: 'Daily Spending',
						data: days.map((day) => dailySpending[day] || 0),
						borderColor: '#3B82F6',
						tension: 0.1
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
				}
			}
		});
	}

	function getCategoryName(categoryId: number): string {
		return categories.find((c) => c.id === categoryId)?.name || 'Uncategorized';
	}

	function formatCurrency(amount: number): string {
		return `₦${amount.toLocaleString('en-NG', { minimumFractionDigits: 2 })}`;
	}

	$: if (selectedMonth) {
		setTimeout(generateCharts, 0);
	}
</script>

<div class="flex min-h-screen">
	<Navigation />

	<main class="flex-1 bg-gray-50 p-8 pt-20 lg:pt-8">
		<header class="mb-8">
			<div class="pl-16 lg:pl-0">
				<h1 class="text-2xl font-bold text-gray-800">Spending Reports</h1>
				<p class="text-gray-600">Analyze your spending patterns</p>
			</div>
		</header>

		<!-- Monthly Summary -->
		<div class="mb-8 grid grid-cols-1 gap-6 md:grid-cols-3">
			{#if selectedMonth}
				{@const monthTransactions = transactions.filter((t) => {
					const [year, month] = selectedMonth.split('-').map(Number);
					const tDate = new Date(t.date);
					return tDate.getMonth() === month - 1 && tDate.getFullYear() === year;
				})}
				{@const income = monthTransactions
					.filter((t) => t.type === 'income')
					.reduce((sum, t) => sum + t.amount, 0)}
				{@const expenses = monthTransactions
					.filter((t) => t.type === 'expense')
					.reduce((sum, t) => sum + t.amount, 0)}
				{@const savings = income - expenses}

				<div class="rounded-lg bg-white p-6 shadow-sm">
					<h3 class="text-sm font-medium text-gray-600">Total Income</h3>
					<p class="mt-2 text-2xl font-bold text-green-500">{formatCurrency(income)}</p>
				</div>
				<div class="rounded-lg bg-white p-6 shadow-sm">
					<h3 class="text-sm font-medium text-gray-600">Total Expenses</h3>
					<p class="mt-2 text-2xl font-bold text-red-500">{formatCurrency(expenses)}</p>
				</div>
				<div class="rounded-lg bg-white p-6 shadow-sm">
					<h3 class="text-sm font-medium text-gray-600">Net Savings</h3>
					<p
						class="mt-2 text-2xl font-bold"
						class:text-green-500={savings >= 0}
						class:text-red-500={savings < 0}
					>
						{formatCurrency(savings)}
					</p>
				</div>
			{/if}
		</div>

		<!-- Charts -->
		<div class="grid grid-cols-1 gap-6 lg:grid-cols-2">
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h2 class="mb-4 text-xl font-semibold text-gray-800">Category Breakdown</h2>
				<div class="relative mx-auto aspect-square w-full max-w-[400px]">
					<canvas bind:this={reportCanvas}></canvas>
				</div>
			</div>
			<div class="rounded-lg bg-white p-6 shadow-sm">
				<h2 class="mb-4 text-xl font-semibold text-gray-800">Daily Spending Trend</h2>
				<div class="relative h-[300px]">
					<canvas bind:this={trendCanvas}></canvas>
				</div>
			</div>
		</div>
	</main>
</div>
