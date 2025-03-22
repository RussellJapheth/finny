<script lang="ts">
	import { page } from '$app/state';
	let isMenuOpen = false;
	let isSettingsOpen = false;

	function toggleMenu() {
		isMenuOpen = !isMenuOpen;
	}

	function exportAllData() {
		const data = {
			transactions: JSON.parse(localStorage.getItem('transactions') || '[]'),
			categories: JSON.parse(localStorage.getItem('categories') || '[]'),
			accounts: JSON.parse(localStorage.getItem('accounts') || '[]'),
			budgets: JSON.parse(localStorage.getItem('budgets') || '[]'),
			exportDate: new Date().toISOString()
		};

		const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
		const url = window.URL.createObjectURL(blob);
		const link = document.createElement('a');
		link.setAttribute('href', url);
		link.setAttribute('download', `finny-backup-${new Date().toISOString().split('T')[0]}.json`);
		link.style.visibility = 'hidden';
		document.body.appendChild(link);
		link.click();
		document.body.removeChild(link);
	}

	function importData(event: Event) {
		const input = event.target as HTMLInputElement;
		const file = input?.files?.[0];
		if (!file) return;

		const reader = new FileReader();
		reader.onload = (e) => {
			try {
				const data = JSON.parse(e.target?.result as string);

				if (!data.transactions || !data.categories || !data.accounts) {
					alert('Invalid backup file format');
					return;
				}

				localStorage.setItem('transactions', JSON.stringify(data.transactions));
				localStorage.setItem('categories', JSON.stringify(data.categories));
				localStorage.setItem('accounts', JSON.stringify(data.accounts));
				if (data.budgets) {
					localStorage.setItem('budgets', JSON.stringify(data.budgets));
				}

				window.location.reload();
			} catch (error) {
				alert('Error importing data. Please check the file format.');
			}
		};
		reader.readAsText(file);
	}
</script>

<!-- Mobile Menu Button (visible on small screens) -->
<button
	class="fixed top-4 left-4 z-50 rounded-md bg-gray-800 p-2 text-white lg:hidden"
	on:click={toggleMenu}
	aria-label="Toggle menu"
>
	{#if isMenuOpen}
		<svg
			xmlns="http://www.w3.org/2000/svg"
			class="h-6 w-6"
			fill="none"
			viewBox="0 0 24 24"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M6 18L18 6M6 6l12 12"
			/>
		</svg>
	{:else}
		<svg
			xmlns="http://www.w3.org/2000/svg"
			class="h-6 w-6"
			fill="none"
			viewBox="0 0 24 24"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M4 6h16M4 12h16M4 18h16"
			/>
		</svg>
	{/if}
</button>

<!-- Overlay for mobile menu -->
{#if isMenuOpen}
	<div class="bg-opacity-50 fixed inset-0 z-40 bg-black md:hidden" on:click={toggleMenu}></div>
{/if}

<!-- Navigation Sidebar -->
<nav
	class={`
	fixed inset-y-0 left-0 z-40 w-64 transform bg-gray-900 p-8 text-white transition-transform duration-200 ease-in-out
	md:relative md:translate-x-0
	${isMenuOpen ? 'translate-x-0' : '-translate-x-full'}
`}
>
	<div class="mb-8">
		<h2 class="text-2xl font-bold text-green-500">Finny</h2>
	</div>
	<ul class="space-y-4">
		<li class={page.url.pathname === '/' ? 'rounded-md bg-gray-800' : ''}>
			<a
				href="/"
				class="block rounded-md px-4 py-2 transition-colors hover:bg-gray-800"
				on:click={() => (isMenuOpen = false)}
			>
				Dashboard
			</a>
		</li>
		<li class={page.url.pathname === '/accounts' ? 'rounded-md bg-gray-800' : ''}>
			<a
				href="/accounts"
				class="block rounded-md px-4 py-2 transition-colors hover:bg-gray-800"
				on:click={() => (isMenuOpen = false)}
			>
				Accounts
			</a>
		</li>
		<li class={page.url.pathname === '/transactions' ? 'rounded-md bg-gray-800' : ''}>
			<a
				href="/transactions"
				class="block rounded-md px-4 py-2 transition-colors hover:bg-gray-800"
				on:click={() => (isMenuOpen = false)}
			>
				Transactions
			</a>
		</li>
		<li class={page.url.pathname === '/budget' ? 'rounded-md bg-gray-800' : ''}>
			<a href="/budget" class="block rounded-md px-4 py-2 transition-colors hover:bg-gray-800"
				>Budget</a
			>
		</li>
		<li class={page.url.pathname === '/categories' ? 'rounded-md bg-gray-800' : ''}>
			<a
				href="/categories"
				class="block rounded-md px-4 py-2 transition-colors hover:bg-gray-800"
				on:click={() => (isMenuOpen = false)}
			>
				Categories
			</a>
		</li>
		<li class={page.url.pathname === '/reports' ? 'rounded-md bg-gray-800' : ''}>
			<a
				href="/reports"
				class="block rounded-md px-4 py-2 transition-colors hover:bg-gray-800"
				on:click={() => (isMenuOpen = false)}
			>
				Reports
			</a>
		</li>
		<li>
			<button
				class="block w-full rounded-md px-4 py-2 text-left transition-colors hover:bg-gray-800"
				on:click={() => (isSettingsOpen = true)}
			>
				Settings
			</button>
		</li>
	</ul>
</nav>

<!-- Add the Settings Modal -->
{#if isSettingsOpen}
	<div class="bg-opacity-50 fixed inset-0 z-50 flex items-center justify-center bg-black">
		<div class="mx-4 w-full max-w-md rounded-lg bg-white p-6 shadow-xl">
			<div class="mb-4 flex items-center justify-between">
				<h2 class="text-xl font-bold text-gray-800">Settings</h2>
				<button class="text-gray-500 hover:text-gray-700" on:click={() => (isSettingsOpen = false)}>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-6 w-6"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M6 18L18 6M6 6l12 12"
						/>
					</svg>
				</button>
			</div>

			<div class="space-y-4">
				<div class="rounded-lg border border-gray-200 p-4">
					<h3 class="mb-2 font-medium text-gray-800">Data Management</h3>
					<div class="flex flex-col gap-2">
						<button
							on:click={exportAllData}
							class="rounded-md bg-blue-500 px-4 py-2 text-white transition-colors hover:bg-blue-600"
						>
							Export All Data
						</button>

						<label
							class="cursor-pointer rounded-md bg-green-500 px-4 py-2 text-center text-white transition-colors hover:bg-green-600"
						>
							Import Data
							<input type="file" accept=".json" on:change={importData} class="hidden" />
						</label>
					</div>
				</div>

				<!-- Add more settings sections here -->
			</div>

			<div class="mt-6 flex justify-end">
				<button
					class="rounded-md bg-gray-200 px-4 py-2 text-gray-700 transition-colors hover:bg-gray-300"
					on:click={() => (isSettingsOpen = false)}
				>
					Close
				</button>
			</div>
		</div>
	</div>
{/if}
