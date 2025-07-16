<script>
	import { onMount } from 'svelte';

	let taskData = [];
	let removedTaskData = [];

	// Load tasks from localStorage when the component mounts
	onMount(() => {
		taskData = JSON.parse(localStorage.getItem('data')) || [];
	});

	const addTask = (event) => {
		event.preventDefault();

		const taskTitle = document.getElementById('todo-input').value;
		const taskLabel = document.getElementById('label-input').value;
		const labelColor = document.getElementById('color-input').value;

		if (!taskTitle || !taskLabel) {
			alert('Please enter a task and a label');
			return;
		}

		const taskObj = {
			id: `${taskTitle.toLowerCase().split(' ').join('-')}-${Date.now()}`,
			title: taskTitle,
			label: taskLabel,
			labelColor: labelColor,
			done: false
		};

		taskData = [taskObj, ...taskData]; // Add new task at the start
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const toggleTask = (taskId) => {
		taskData = taskData.map((task) => (task.id === taskId ? { ...task, done: !task.done } : task));
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const deleteTask = (taskId) => {
		taskData = taskData.filter((task) => task.id !== taskId);
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const deleteToDoList = () => {
		removedTaskData = [...taskData];
		taskData = [];
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const restoredRemovedTasks = () => {
		taskData = [...removedTaskData];
		removedTaskData = [];
		localStorage.setItem('data', JSON.stringify(taskData));
	};
</script>

<svelte:head>
	<title>My to-do list - Get the work done!</title>
</svelte:head>

<header class="font-Nunito-Sans pt-5">
	<h1 class="my-4 text-3xl font-bold">My to-do list</h1>
</header>

<div class="todo-form font-Nunito">
	<form on:submit={addTask}>
		<div>
			<input
				class="mr-2 h-[35px] rounded border px-5"
				type="text"
				placeholder="Add a new task"
				id="todo-input"
			/>
			<input
				class="mr-2 h-[35px] rounded border px-5"
				type="text"
				placeholder="Add a label"
				id="label-input"
			/>
			<label for="color-input">Label color:</label>
			<input
				class="cursor-pointer rounded border px-1"
				type="color"
				id="color-input"
				value="#19D28B"
			/>
		</div>
		<div>
			<button
				class="bg-checkmark-purple mt-2 h-[35px] rounded border-2 px-4 font-bold text-white hover:cursor-pointer hover:bg-blue-700"
				type="submit">Add</button
			>
		</div>
	</form>

	<ul
		class="todo-list my-6 max-w-[600px] list-none rounded-[10px] border border-gray-300 max-md:max-w-[400px]"
	>
		{#each taskData as task (task.id)}
			<li
				class="flex items-center justify-between border-b border-[#ccc] p-4 font-semibold first:rounded-t-[10px] last:rounded-b-[10px] last:border-none {task.done
					? 'bg-[#f3f3f3]'
					: ''}"
			>
				<label class="relative flex cursor-pointer items-center">
					<input
						type="checkbox"
						class="peer hidden"
						checked={task.done}
						on:change={() => toggleTask(task.id)}
					/>
					<div
						class="peer-checked:bg-checkmark-purple peer-checked:border-checkmark-purple flex h-5 w-5 items-center justify-center rounded-full border-2 border-black text-[12px] text-transparent peer-checked:text-white"
					>
						✔
					</div>
				</label>
				<span class="task pl-2 {task.done ? 'text-black line-through' : ''}">{task.title}</span>
				<span
					class="ml-3 rounded px-3 py-1 text-xs font-semibold text-white"
					style="background-color:{task.labelColor}">{task.label}</span
				>
				<button
					class="ml-auto h-[25px] w-[25px] rounded-full border-2 border-red-600 bg-white text-[12px] font-bold text-red-600 hover:cursor-pointer"
					on:click={() => deleteTask(task.id)}>X</button
				>
			</li>
		{/each}
	</ul>
</div>

<div class="flex gap-x-4">
	<button
		class="rounded border-2 bg-[#ff0000] px-5 py-2 text-sm font-semibold text-white hover:cursor-pointer hover:bg-red-700"
		on:click={deleteToDoList}>Clear all items</button
	>
	{#if removedTaskData.length > 0}
		<button
			class="rounded border-2 bg-blue-400 px-5 py-2 text-sm font-semibold text-white hover:cursor-pointer hover:bg-blue-600"
			on:click={restoredRemovedTasks}>Restore lists</button
		>
	{/if}
</div>
