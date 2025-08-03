<script>
	import { onMount, onDestroy } from 'svelte';
	import { fade } from 'svelte/transition';
	import { flip } from 'svelte/animate';

	import CheckCircle from '$lib/icons/CheckCircle.svelte';
	import CheckIcon from '$lib/icons/CheckIcon.svelte';
	import ChevronUp from '$lib/icons/ChevronUp.svelte';
	import ChevronDown from '$lib/icons/ChevronDown.svelte';
	import DeleteIcon from '$lib/icons/DeleteIcon.svelte';
	import EditIcon from '$lib/icons/EditIcon.svelte';
	import TrashIcon from '$lib/icons/TrashIcon.svelte';
	import LabelIcon from '$lib/icons/LabelIcon.svelte';

	// Array to store tasks
	let taskData = $state([]);
	// Array to store removed tasks
	let removedTaskData = $state([]);

	// Variables for editing tasks
	let editingTaskId = $state(null);
	let editTitle = $state('');
	let editLabel = $state('');

	// State for dropdown and selected label
	let dropdown = $state();
	let showDropdown = $state(false);
	let open = $state(false);
	let selectedLabel = $state('No label');
	let labels = [
		{ name: 'No label', color: '#f3f4f6' },
		{ name: 'Orange', color: '#fed7aa' },
		{ name: 'Green', color: '#bbf7d0' },
		{ name: 'Blue', color: '#bfdbfe' },
		{ name: 'Purple', color: '#ddd6fe' },
		{ name: 'Pink', color: '#fbcfe8' },
		{ name: 'Red', color: '#fecaca' }
	];

	// Load tasks from localStorage when the component mounts
	onMount(() => {
		taskData = JSON.parse(localStorage.getItem('data')) || [];
		removedTaskData = JSON.parse(localStorage.getItem('removed')) || [];
		document.addEventListener('click', handleClickOutside);
	});

	const addTask = (event) => {
		event.preventDefault();

		const taskTitle = document.getElementById('todo-input').value;
		const taskLabel = document.getElementById('label-input').value;
		// const labelColor = document.getElementById('color-input').value;
		const labelColor = labels.find((l) => l.name === selectedLabel)?.color;

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

		// Clear form inputs after adding task
		document.getElementById('todo-input').value = '';
		document.getElementById('label-input').value = '';
		// document.getElementById('color-input').value = '';
	};

	const selectLabel = (label) => {
		selectedLabel = label.name;
		showDropdown = false;
		open = false;
	};

	const handleClickOutside = (event) => {
		if (dropdown && !dropdown.contains(event.target)) {
			showDropdown = false;
			open = false;
		}
	};

	const toggleTask = (taskId) => {
		taskData = taskData.map((task) => (task.id === taskId ? { ...task, done: !task.done } : task));
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const moveTaskUp = (index) => {
		if (index > 0) {
			// Copy the array
			const newTasks = [...taskData];
			// Swap the task with the one above it
			[newTasks[index - 1], newTasks[index]] = [newTasks[index], newTasks[index - 1]];
			taskData = newTasks;
			localStorage.setItem('data', JSON.stringify(taskData));
		}
	};

	const moveTaskDown = (index) => {
		if (index < taskData.length - 1) {
			const newTasks = [...taskData];
			[newTasks[index], newTasks[index + 1]] = [newTasks[index + 1], newTasks[index]];
			taskData = newTasks;
			localStorage.setItem('data', JSON.stringify(taskData));
		}
	};

	const editTask = (taskId) => {
		const task = taskData.find((t) => t.id === taskId);
		if (task) {
			editingTaskId = taskId;
			editTitle = task.title;
			editLabel = task.label;
		}
	};

	const saveEditTask = (taskId) => {
		taskData = taskData.map((task) =>
			task.id === taskId ? { ...task, title: editTitle, label: editLabel } : task
		);
		localStorage.setItem('data', JSON.stringify(taskData));
		editingTaskId = null;
	};

	const deleteTask = (taskId) => {
		taskData = taskData.filter((task) => task.id !== taskId);
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const deleteToDoList = () => {
		removedTaskData = [...taskData];
		localStorage.setItem('removed', JSON.stringify(removedTaskData)); // Save removed
		taskData = [];
		localStorage.setItem('data', JSON.stringify(taskData));
	};

	const restoredRemovedTasks = () => {
		taskData = [...removedTaskData];
		localStorage.setItem('data', JSON.stringify(taskData));
		removedTaskData = [];
		localStorage.setItem('removed', JSON.stringify(removedTaskData)); // Clear removed
		console.log('Restored removed tasks:', taskData);
	};
</script>

<svelte:head>
	<title>My to-do list - Get the work done!</title>
</svelte:head>

<header class="font-Nunito-Sans pt-5">
	<h1 class="my-4 text-3xl font-bold">My to-do list</h1>
</header>

<!-- <div class="todo-form font-Nunito">
	<form onsubmit={addTask}>
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
						onchange={() => toggleTask(task.id)}
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
					onclick={() => deleteTask(task.id)}>X</button
				>
			</li>
		{/each}
	</ul>
</div>

<div class="flex gap-x-4">
	<button
		class="rounded border-2 bg-[#ff0000] px-5 py-2 text-sm font-semibold text-white hover:cursor-pointer hover:bg-red-700"
		onclick={deleteToDoList}>Clear all items</button
	>
	{#if removedTaskData.length > 0}
		<button
			class="rounded border-2 bg-blue-400 px-5 py-2 text-sm font-semibold text-white hover:cursor-pointer hover:bg-blue-600"
			onclick={restoredRemovedTasks}>Restore lists</button
		>
	{/if}
</div> -->

<div class="font-Nunito mt-6 rounded-lg bg-white p-8 shadow-lg">
	<div class="flex items-center gap-x-2">
		<CheckCircle class="h-6 w-6 text-green-500" />
		<h2 class="py-2 text-2xl font-bold">Add a new task</h2>
	</div>
	<form onsubmit={addTask}>
		<div class="flex items-center gap-x-2 py-2">
			<input
				class="mr-2 w-full rounded-lg border border-gray-300 px-5"
				type="text"
				placeholder="What needs to be done?"
				id="todo-input"
			/>
			<div class="">
				<button
					class="inline-flex items-center justify-center rounded-lg border-2 bg-black px-6 py-2 font-bold text-white hover:cursor-pointer hover:bg-gray-700"
					type="submit"
					><svg
						xmlns="http://www.w3.org/2000/svg"
						width="24"
						height="24"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2"
						stroke-linecap="round"
						stroke-linejoin="round"
						class="lucide lucide-plus mr-2 h-4 w-4"
						><path d="M5 12h14"></path><path d="M12 5v14"></path></svg
					>Add</button
				>
			</div>
		</div>

		<div class="mt-2 flex w-full flex-col gap-2 md:flex-row">
			<div class="flex items-center gap-2">
				<LabelIcon class="inline-block h-4 w-4 text-gray-500" />
				<div class="relative w-48 text-sm" bind:this={dropdown}>
					<button
						type="button"
						class="flex w-full cursor-pointer items-center justify-between rounded-lg border border-gray-300 bg-white px-3 py-2 text-left"
						onclick={() => {
							showDropdown = !showDropdown;
							open = !open;
						}}
					>
						<span
							class="mr-2 inline-block h-3 w-3 rounded-full"
							style="background-color: {labels.find((l) => l.name === selectedLabel)?.color}"
						></span>
						{selectedLabel}
						<svg
							class="ml-auto h-4 w-4 transform text-gray-400 transition-transform duration-300"
							class:rotate-180={open}
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M19 9l-7 7-7-7"
							/>
						</svg>
					</button>
					{#if showDropdown}
						<ul
							class="absolute z-10 mt-2 w-full max-w-[184px] space-y-1 rounded-lg border border-gray-200 bg-white py-2 shadow-lg"
							transition:fade={{ duration: 200 }}
						>
							{#each labels as label}
								<li
									class="flex cursor-pointer items-center px-8 py-2 hover:bg-gray-100"
									onclick={() => selectLabel(label)}
								>
									<span
										class="mr-2 inline-block h-3 w-3 rounded-full"
										style="background-color: {label.color}"
									></span>
									{label.name}
								</li>
							{/each}
						</ul>
					{/if}
				</div>
			</div>
			<input
				class="w-full rounded-lg border border-gray-300 bg-white px-3 py-2 text-left text-sm md:w-48"
				type="text"
				placeholder="Label name"
				id="label-input"
			/>
		</div>
	</form>
</div>

<!-- If no tasks -->
{#if taskData.length === 0}
	<div class="font-Nunito my-6 rounded-lg bg-white p-8 shadow-lg">
		<CheckCircle class="mx-auto h-12 w-12 text-[#D1D5DB]" />
		<p class="py-2 text-center text-xl font-semibold text-gray-500">No tasks yet</p>
		<p class="text-center text-gray-400">Add a task above to get started</p>
	</div>
{:else}
	<!-- Progress bar -->
	<!-- If task -->
	<div class="font-Nunito my-6 rounded-lg bg-white shadow-lg">
		<ul class="todo-list">
			{#each taskData as task, i (task.id)}
				<li
					class="flex items-center justify-between gap-2 border-b border-[#ccc] p-4 font-semibold first:rounded-t-[10px] last:rounded-b-[10px] last:border-none {task.done
						? 'bg-gray-100'
						: ''}"
					animate:flip
				>
					<input
						type="checkbox"
						class="cursor-pointer text-green-500 focus:ring-0"
						id="toggle-tasks"
						checked={task.done}
						onchange={() => toggleTask(task.id)}
					/>
					<div class="flex-1">
						{#if editingTaskId === task.id}
							<input class="mb-1 w-full rounded border px-2 py-1" bind:value={editTitle} />
							<input class="w-full rounded border px-2 py-1" bind:value={editLabel} />
						{:else}
							<label class={task.done ? 'text-gray-500 line-through' : ''}>{task.title}</label>
							<span
								class="ml-3 rounded px-3 py-1 text-xs font-semibold text-white"
								style="background-color:{task.labelColor}">{task.label}</span
							>
						{/if}
					</div>
					<div class="flex gap-2">
						{#if i > 0}
							<button
								onclick={() => moveTaskUp(i)}
								class="p-2 hover:cursor-pointer hover:rounded hover:bg-gray-100"
							>
								<ChevronUp class="h-4 w-4 text-gray-500" />
							</button>
						{/if}
						{#if i < taskData.length - 1}
							<button
								onclick={() => moveTaskDown(i)}
								class="p-2 hover:cursor-pointer hover:rounded hover:bg-gray-100"
							>
								<ChevronDown class="h-4 w-4 text-gray-500" />
							</button>
						{/if}
						<!-- <button
							onclick={moveTaskDown}
							class="p-2 hover:cursor-pointer hover:rounded hover:bg-gray-100"
						>
							<ChevronDown class="h-4 w-4 text-gray-500" />
						</button> -->
						{#if editingTaskId === task.id}
							<button
								onclick={() => saveEditTask(task.id)}
								class="p-2 hover:cursor-pointer hover:rounded hover:bg-green-50"
							>
								<CheckIcon class="h-4 w-4 text-green-500" />
							</button>
							<button
								onclick={() => (editingTaskId = null)}
								class="p-2 hover:cursor-pointer hover:rounded hover:bg-gray-100"
							>
								✕
							</button>
						{:else}
							<button
								onclick={() => editTask(task.id)}
								class="p-2 hover:cursor-pointer hover:rounded hover:bg-blue-50"
							>
								<EditIcon class="h-4 w-4 text-blue-500" />
							</button>
						{/if}
						<button
							onclick={() => deleteTask(task.id)}
							class="p-2 hover:cursor-pointer hover:rounded hover:bg-red-50"
						>
							<TrashIcon class="h-4 w-4 text-red-500" />
						</button>
					</div>
				</li>
			{/each}
		</ul>
	</div>

	<!-- Clear all items -->
	<div class="flex gap-x-4">
		<button
			class="rounded-lg border-2 bg-[#ff0000] px-5 py-2 text-sm font-semibold text-white hover:cursor-pointer hover:bg-red-700"
			onclick={deleteToDoList}>Clear all items</button
		>
	</div>
{/if}
{#if removedTaskData.length > 0}
	<button
		class="rounded-lg border-2 bg-green-500 px-5 py-2 text-sm font-semibold text-white hover:cursor-pointer hover:bg-green-600"
		onclick={restoredRemovedTasks}>Restore lists</button
	>
{/if}
