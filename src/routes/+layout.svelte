<script lang="ts">
	import '../app.css';
	let { children } = $props();
</script>

<div class=" mx-auto box-border max-w-2xl px-6">
	{@render children()}
</div>
