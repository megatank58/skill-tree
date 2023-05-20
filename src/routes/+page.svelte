<script lang="ts">
	import { browser } from '$app/environment';

	class Data {
		constructor(public cols: number, public rows: number, public nodes: Node[]) {}

		toString() {
			return JSON.stringify(this);
		}

		static from(d: string) {
			let object = JSON.parse(d);

			let data = new this(object.cols, object.rows, object.nodes);

			return data;
		}
	}

	class Node {
		constructor(
			public col: number,
			public row: number,
			public svg: string,
			public tasks: Task[] = [],
			public finished: boolean = false
		) {}
	}

	class Task {
		constructor(public name: string, public finished: boolean = false) {}
	}

	let generated = browser ? localStorage.getItem('table') : false;
	let cols: any[];
	let rows: any[];
	let id: string;
	let data: Data;
	let popup = false;

	if (generated) {
		data = Data.from(generated);

		cols = new Array(data?.cols);
		rows = new Array(data?.rows);

		console.log(data);
	}

	function save(d: Data) {
		d.nodes.sort();

		data = d;

		localStorage.setItem('table', data.toString());
	}

	function generate() {
		let col = parseInt((document.getElementById('cols') as HTMLInputElement).value);
		let row = parseInt((document.getElementById('rows') as HTMLInputElement).value);

		data = new Data(col, row, []);

		save(data);

		generated = true;

		cols = new Array(data?.cols);
		rows = new Array(data?.rows);
	}

	function addNode() {
		let col = parseInt(id.split(' ')[0]);
		let row = parseInt(id.split(' ')[1]);

		let list = document.getElementById('tasks')?.getElementsByTagName('input')!;

		let tasks: Task[] = [];

		for (let item of list) {
			tasks.push(new Task(item.value));
		}

		data.nodes.push(
			new Node(col, row, (document.getElementById('svg') as HTMLInputElement).value, tasks)
		);

		save(data);

		id = '';
		popup = false;
	}

	function addTask() {
		let element = document.createElement('div');
		element.id = 'tasks';
		element.innerHTML = `
						<input
							type="text"
							placeholder="task"
							class="input input-bordered input-info w-full max-w-xs mt-2"
						/>`;
		document.getElementById('tasks')?.appendChild(element);
	}

	function toggleFinished(id: string) {
		let col = parseInt(id.split(' ')[0]);
		let row = parseInt(id.split(' ')[1]);

		let node = data.nodes.find((node) => col === node.col && row === node.row)!;
		let index = data.nodes.indexOf(node);

		node.finished = !node.finished;

		data.nodes[index] = node;

		save(data);
	}

	function toggleTask(id: string) {
		let col = parseInt(id.split(' ')[1]);
		let row = parseInt(id.split(' ')[2]);
		let task = parseInt(id.split(' ')[3]);

		let node = data.nodes.find((node) => col === node.col && row === node.row)!;
		let index = data.nodes.indexOf(node);

		data.nodes[index].tasks[task].finished = !data.nodes[index].tasks[task].finished;

		save(data);
	}

	function position(node: SVGLineElement) {
		let col = node.id.split(' ')[0];
		let row = node.id.split(' ')[1];
		node.setAttribute(
			'x1',
			document.getElementById(`${col} ${row}`)!.getBoundingClientRect().left.toString()
		);
		node.setAttribute(
			'x2',
			document.getElementById(`origin`)!.getBoundingClientRect().left.toString()
		);
		node.setAttribute(
			'y1',
			document.getElementById(`${col} ${row}`)!.getBoundingClientRect().top.toString()
		);
		node.setAttribute(
			'y2',
			document.getElementById(`origin`)!.getBoundingClientRect().top.toString()
		);
	}
</script>

{#if generated}
	<div
		class="w-full min-h-screen grid"
		style="
		grid-template-columns: repeat({data.cols} ,minmax(0, 1fr));
		grid-template-rows: repeat({data.rows} ,minmax(0, 1fr));
	"
	>
		{#if popup}
			<input type="checkbox" id="popop-modal" checked={true} class="modal-toggle" />
			<div class="modal">
				<div class="modal-box">
					<h3 class="font-bold text-lg">Enter SVG</h3>
					<input
						type="text"
						id="svg"
						placeholder="svg"
						class="input input-bordered input-primary w-full max-w-xs m-2"
					/>
					<h3 class="text-sm font-light">Enter tasks (optional)</h3>
					<div id="tasks">
						<input
							type="text"
							placeholder="task"
							class="input input-bordered input-info w-full max-w-xs mt-2"
						/>
					</div>
					<div class="flex">
						<button
							class="ml-auto btn"
							on:click={() => {
								addTask();
							}}>Add</button
						>
					</div>
					<div class="modal-action">
						<button
							class="btn btn-primary"
							id="popup"
							on:click={() => {
								addNode();
							}}>Confirm</button
						>
					</div>
				</div>
			</div>
		{/if}
		{#each cols as _, i}
			{#each rows as _, j}
				<div class="border-l-2 border-b-2 border-base-200 group flex justify-center items-center">
					{#if data.nodes.find((node) => i == node.col && j == node.row)}
					<div class="relative">
						<div class="hidden group-hover:block bg-base-300 rounded absolute top-full left-full element-inside-grid">
							{#each data.nodes.find((node) => i == node.col && j == node.row).tasks as task, k}
								<div class="flex flex-col">
									<div class="form-control">
										<label class="cursor-pointer label">
											<span class="label-text m-2">{task.name}</span>
											<input
												id={`C ${i} ${j} ${k}`}
												type="checkbox"
												class="toggle toggle-primary ml-auto"
												checked={task.finished}
												on:change={({ currentTarget }) => {
													toggleTask(currentTarget.id);
												}}
											/>
										</label>
									</div>
								</div>
							{/each}
						</div>
					</div>
						<button
							id={`${i} ${j}`}
							class={data.nodes.find((node) => i == node.col && j == node.row)?.finished
								? 'btn btn-success mask mask-hexagon-2 min-h-fit h-auto'
								: 'btn btn-base-300 mask mask-hexagon-2 min-h-fit h-auto'}
							on:click={({ currentTarget }) => {
								toggleFinished(currentTarget.id);
							}}
						>
							{@html data.nodes.find((node) => i == node.col && j == node.row)?.svg}
						</button>
					{:else if Math.floor(cols.length / 2) == i && Math.floor(rows.length / 2) == j}
						<div id="origin">
							<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24"
								><g fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
									><path d="M0 0h24v24H0z" /><path
										fill="currentColor"
										d="M12 7a5 5 0 1 1-4.995 5.217L7 12l.005-.217A5 5 0 0 1 12 7z"
									/></g
								></svg
							>
						</div>
					{:else}
						<div class="hidden group-hover:block">
							<button
								id={`${i} ${j}`}
								class="btn btn-base-300 mask mask-hexagon-2 text-xl min-h-fit h-auto"
								on:click={({ currentTarget }) => {
									popup = true;
									id = currentTarget.id;
								}}
							>
								+
							</button>
						</div>
					{/if}
				</div>
			{/each}
		{/each}
	</div>
{:else}
	<input type="checkbox" id="generate-modal" checked={true} class="modal-toggle" />
	<div class="modal">
		<div class="modal-box">
			<h3 class="font-bold text-lg">Enter number of columns and rows</h3>
			<input
				type="text"
				id="cols"
				placeholder="columns"
				class="input input-bordered input-primary w-full max-w-xs mt-2"
			/>
			<input
				type="text"
				id="rows"
				placeholder="rows"
				class="input input-bordered input-primary w-full max-w-xs mt-2"
			/>
			<div class="modal-action">
				<button
					class="btn btn-primary"
					on:click={() => {
						generate();
					}}>Generate!</button
				>
			</div>
		</div>
	</div>
{/if}
