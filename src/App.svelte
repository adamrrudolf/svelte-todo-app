<script>
	import {db} from './firebase';
	import {collectionData} from 'rxfire/firestore';
	import {startWith} from 'rxjs/operators';
	import { fromEvent, Subject } from 'rxjs';
	import { debounceTime, map } from 'rxjs/operators';
	import { onMount, onDestroy } from 'svelte';

	const onEdit$ = new Subject()
	onMount(() => {
		onEdit$.pipe(
			debounceTime(500),
			map(([event, id]) => [event.target.value, id]),
		).subscribe(updateName);
	});
	onDestroy(() => {
		onEdit$.unsubscribe();
	})

	const query = db.collection("list").orderBy("pos", "asc");
	const todos = collectionData(query, 'id').pipe(startWith([]));
	let newItemText = '';
	const addItem = () => {
		let pos = new Date().getTime();
	  db.collection("list").add({name: newItemText, pos})
	}
	const deleteItem = (id) => {
		db.collection("list").doc(id).delete();
	}
	const updateName = ([name, id]) => {
		db.collection("list").doc(id).update({name});
	}
	let isEditing;
	const toggleEdit = () => {
		isEditing = !isEditing;
	}
	
</script>

<main>
	<h1>List 3</h1>
	Add item:
	<input type="text" bind:value={newItemText}/>
	<button on:click={addItem}>Primary</button>
	<ul>
	{#each $todos as todo}
		<li>
			<button on:click={() => deleteItem(todo.id)}>X</button>
			{#if isEditing}
				<input id={todo.id} type="text" value={todo.name} on:keyup={(e) => onEdit$.next([e, todo.id])} />
			{:else}
				<span>{todo.name}</span>
			{/if} 
			<button on:click={toggleEdit}>e</button>
		</li>
	{/each}
	</ul>
</main>
