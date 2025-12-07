<script lang="ts">
	import favicon from "$lib/assets/favicon.jpg";
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Lexend:wght@400&display=swap"
		rel="stylesheet"
	/>
	<title>loops and captions</title>
</svelte:head>

{@render children()}
