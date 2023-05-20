<script>
	import '../app.css';
</script>

<div class="bg-base-100">
	<div class="navbar bg-transparent shadow-xl mb-2">
		<a class="btn btn-ghost normal-case" href="/">SkillTree</a>
		<button
			class="btn btn-error text-white ml-auto"
			on:click={() => {
				localStorage.clear();
				window.location = window.location;
			}}
			><svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24"
				><path
					fill="currentColor"
					d="M15.14 3c-.51 0-1.02.2-1.41.59L2.59 14.73c-.78.77-.78 2.04 0 2.83L5.03 20h7.66l8.72-8.73c.79-.77.79-2.04 0-2.83l-4.85-4.85c-.39-.39-.91-.59-1.42-.59M17 18l-2 2h7v-2"
				/></svg
			></button
		>
	</div>
	<slot />
</div>
