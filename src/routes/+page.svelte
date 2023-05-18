<script lang="ts">
	import { browser } from '$app/environment';

	class Data {
		public cols: number;
		public rows: number;
		public nodes: Node[];

		constructor(cols: number, rows: number) {
			this.cols = cols;
			this.rows = rows;
			this.nodes = [];
		}

		toString() {
			return `${this.cols} ${this.rows},${this.nodes.map((node) => `${node.toString()},`)}`;
		}

		static from(d: string) {
			let table = d.split(' ');

			let cols = parseInt(table[0]);
			let rows = parseInt(table[1]);

			let object = new this(cols, rows);

			let nodes = d.split(',');
			nodes.splice(0, 1);

			nodes.map((node) => {
				let dimensions = node.split(' ');
				let col = parseInt(dimensions[0]);
				let row = parseInt(dimensions[1]);

				if (!isNaN(col) && !isNaN(row)) {
					let finished = dimensions[2] === 'true';

					dimensions.splice(0, 3);

					let str = dimensions.join(' ');

					let n = new Node(col, row, str);

					n.finished = finished;

					object.nodes.push(n);
				}
			});

			return object;
		}
	}

	class Node {
		public col: number;
		public row: number;
		public finished: boolean;
		public svg: string;

		constructor(col: number, row: number, svg: string) {
			this.col = col;
			this.row = row;
			this.finished = false;
			this.svg = svg;
		}

		toString() {
			return `${this.col} ${this.row} ${this.finished} ${this.svg}`;
		}
	}

	let generated = browser ? localStorage.getItem('table') : false;
	let cols: any[];
	let rows: any[];
	let id: string;
	let data: Data;
	let popup = false;

	if (generated) {
		data = Data.from(generated);

		console.log(data);
	}

	function save(d: Data) {
		data.nodes.sort();

		data = data;

		localStorage.setItem('table', data.toString());
	}

	function generate() {
		let col = parseInt((document.getElementById('cols') as HTMLInputElement).value);
		let row = parseInt((document.getElementById('rows') as HTMLInputElement).value);

		data = new Data(col, row);

		save(data);

		generated = true;

		cols = new Array(data?.cols);
		rows = new Array(data?.rows);
	}

	function addNode() {
		let col = parseInt(id.split(' ')[0]);
		let row = parseInt(id.split(' ')[1]);

		data.nodes.push(new Node(col, row, (document.getElementById('svg') as HTMLInputElement).value));

		save(data);

		id = '';
		popup = false;
	}

	function toggleFinished(id: string) {
		let col = parseInt(id.split(' ')[0]);
		let row = parseInt(id.split(' ')[1]);

		let node = data.nodes.find((node) => col === node.col && row === node.row);
		let index = data.nodes.indexOf(node);

		node.finished = !node.finished;

		data.nodes[index] = node;

		save(data);
	}

	if (generated) {
		cols = new Array(data?.cols);
		rows = new Array(data?.rows);
	}

	function position(node: SVGLineElement) {
		let col = node.id.split(' ')[0];
		let row = node.id.split(' ')[1];
		node.setAttribute(
			'x1',
			document.getElementById(`${col} ${row}`).getBoundingClientRect().left.toString()
		);
		node.setAttribute(
			'x2',
			document.getElementById(`origin`).getBoundingClientRect().left.toString()
		);
		node.setAttribute(
			'y1',
			document.getElementById(`${col} ${row}`).getBoundingClientRect().top.toString()
		);
		node.setAttribute(
			'y2',
			document.getElementById(`origin`).getBoundingClientRect().top.toString()
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
					<h3 class="font-bold text-lg">Enter number of columns and rows</h3>
					<input
						type="text"
						id="svg"
						placeholder="svg"
						class="input input-bordered input-primary w-full max-w-xs mt-2"
					/>
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
				<div class="group flex justify-center items-center">
					{#if data.nodes.find((node) => i == node.col && j == node.row)}
						<button
							id={`${i} ${j}`}
							class={data.nodes.find((node) => i == node.col && j == node.row).finished
								? 'btn btn-success mask mask-hexagon-2 text-4xl mt-1'
								: 'btn btn-base-300 mask mask-hexagon-2 text-4xl mt-1'}
							on:click={({ currentTarget }) => {
								toggleFinished(currentTarget.id);
							}}
						>
							{@html data.nodes.find((node) => i == node.col && j == node.row).svg}
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
								class="btn btn-base-300 mask mask-hexagon-2 text-4xl mt-1"
								on:click={({ currentTarget }) => {
									popup = true;
									id = currentTarget.id;
								}}
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 24 24"
									width="36px"
									height="40px"
									><path
										fill="currentColor"
										d="M12 19q-.425 0-.713-.288T11 18v-5H6q-.425 0-.713-.288T5 12q0-.425.288-.713T6 11h5V6q0-.425.288-.713T12 5q.425 0 .713.288T13 6v5h5q.425 0 .713.288T19 12q0 .425-.288.713T18 13h-5v5q0 .425-.288.713T12 19Z"
									/></svg
								>
							</button>
						</div>
					{/if}
				</div>
			{/each}
		{/each}
	</div>
	<div>
		{#each data.nodes as node}
			<svg><line id={`${node.col} ${node.row}`} use:position stroke="black" /></svg>
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
