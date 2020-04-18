<script>
	const toggleTheme = () => {
		let theme = localStorage.getItem('theme') || 'light';

		if (theme === 'light') {
			document.body.classList.replace('light', 'dark');
			theme = 'dark';
		} else if (theme === 'dark') {
			document.body.classList.replace('dark', 'light');
			theme = 'light';
		}

		localStorage.setItem('theme', theme);
	}
</script>



<wired-card class="header-container">
	<div class="header-content">
		<h2 style="color: #e48f73; font-weight: 400; display: inline;">🍺 CARONA VIRUS 🤢</h2>
		<wired-toggle on:change={toggleTheme} class="theme-toggle"></wired-toggle>
	</div>
	<wired-card class="header-divider" elevation="2"></wired-card>
</wired-card>



<style>
	.theme-toggle {
		float: right;
		--wired-toggle-off-color: #3367d6;
		--wired-toggle-on-color: #ea7075;
	}

	.header-container {
		overflow: hidden;
		display: block;
		margin: -16px;
		width: calc(100% + 32px);
		padding-bottom: 0;
		box-sizing: border-box;
	}

	.header-content {
		text-align: center;
		padding: 1em 0;
	}

	.header-divider {
		color: #e48f73;
		width: 100%;
		margin-left: -8px;
		margin-right: -8px;
	}
</style>
