<script lang="ts">
	import { dev } from '$app/environment';
	import { Replicache, type WriteTransaction } from 'replicache';

	type Todo = {
		id: number;
		name: string;
		deleted: boolean;
		createdAt: string;
		updatedAt: string;
		completed: boolean;
	};

	let todos: Todo[] = $state([]);
	let form_state = $state({ name: '' });

	let name = dev ? 'dev:todos:userId' : 'todos:userId';

	let replicache = new Replicache({
		name,
		licenseKey: import.meta.env.VITE_REPLICACHE_LICENSE,
		mutators: {
			create_todo: async (tx: WriteTransaction, args: Todo) => {
				const key = `todos/${args.id}`;
				await tx.set(key, args);
			},
			toggle_todo: async (tx: WriteTransaction, args: Todo) => {
				const key = `todos/${args.id}`;
				await tx.set(key, args);
			}
		}
	});

	$effect(() => {
		return replicache.subscribe(
			async (tx) => {
				const todos = await tx.scan({ prefix: 'todos/' }).entries().toArray();
				return todos.map(([_, values]) => values as Todo);
			},
			(items: Todo[]) => {
				todos = items;
			}
		);
	});

	function toggle(todo: Todo) {
		replicache.mutate.toggle_todo({
			...todo,
			completed: !todo.completed
		});
	}

	function onsubmit(e: SubmitEvent) {
		e.preventDefault();
		replicache.mutate.create_todo({
			id: new Date().getTime(),
			name: form_state.name,
			deleted: false,
			createdAt: new Date().toISOString(),
			updatedAt: new Date().toISOString(),
			completed: false
		});
		form_state.name = '';
	}
</script>

<form method="POST" {onsubmit}>
	<label for="name">Todo:</label>
	<input type="text" id="name" name="name" bind:value={form_state.name} />
	<button type="submit">+ Add</button>
</form>

<ul>
	{#each todos as todo}
		<li class:completed={todo.completed}>
			<span>{todo.name}</span>
			<button onclick={() => toggle(todo)}>✔</button>
		</li>
	{/each}
</ul>

<style>
	.completed {
		opacity: 0.5;
		text-decoration: line-through;
	}
	li {
		margin-bottom: 5px;
	}
</style>
