<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<div class=" mx-auto box-border max-w-screen-xl px-6">
	{@render children()}
</div>
