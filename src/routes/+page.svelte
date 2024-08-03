<script lang="ts">
	import { dev } from '$app/environment';
	import { Replicache } from 'replicache';

	type Todo = {
		id: number;
		name: string;
		deleted: boolean;
		createdAt: string;
		updatedAt: string;
	};

	let todos: Todo[] = $state([]);
	let form_state = $state({ name: '' });

	let name = dev ? 'dev:todos:userId' : 'todos:userId';

	let replicache = new Replicache({
		name,
		licenseKey: import.meta.env.VITE_REPLICACHE_LICENSE,
		mutators: {}
	});

	function onsubmit(e: Event) {
		e.preventDefault();
	}
</script>

<form method="POST" {onsubmit}>
	<label for="name">Todo:</label>
	<input type="text" id="name" name="name" bind:value={form_state.name} />
	<button type="submit">+ Add</button>
</form>

<ul>
	{#each todos as todo}
		<li>
			<span>{todo.name}</span>
			<button>✔</button>
		</li>
	{/each}
</ul>
