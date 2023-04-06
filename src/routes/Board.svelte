<!-- 
    TODO:
        - Throw confetti when player wins
        - Refactor dynamically setting grid to repeat(${SIZE}, 1fr)
        - Refactor Cell to contain its own row and column
-->
<script lang="ts">
	import Cell from './Cell';

	let SIZE = 8;
	const bombDensity = 0.15;

	let board: Cell[][];

	// make a new SIZExSIZE array
	board = Array.from({ length: SIZE }, () => Array.from({ length: SIZE }, () => new Cell()));

	const randomBool = () => (Math.random() < 1 - bombDensity ? false : true);
	board.forEach((row) => row.forEach((cell) => (cell.isBomb = randomBool())));

	$: {
		if (board.every((row) => row.every((cell) => !cell.isBomb || cell.flagged))) {
			alert("you've won!");
		}
	}

	function onClickCell(i: number, j: number) {
		if (board[i][j].isBomb) {
			alert('game over!');
		}
		board[i][j].clicked = true;
		if (countNeighborBombs(i, j) === 0) {
			propogateEmpty(i, j);
		}
	}

	function onRightClickCell(i: number, j: number) {
		board[i][j].flagged = !board[i][j].flagged;
	}

	function cellContent(i: number, j: number) {
		const n = countNeighborBombs(i, j);

		return n === 0 ? '' : n.toString();
	}

	function countNeighborBombs(i: number, j: number): number {
		return neighbors(i, j).filter((cell) => cell.isBomb).length;
	}

	function neighbors(i: number, j: number): Cell[] {
		const arr = [];
		for (let dx of [-1, 0, 1]) {
			for (let dy of [-1, 0, 1]) {
				let ii = i + dx;
				let jj = j + dy;

				// ignore self
				if (ii === 0 && jj === 0) {
					continue;
				}

				if (insideBounds(ii, jj)) {
					arr.push(board[ii][jj]);
				}
			}
		}

		return arr;
	}

	function insideBounds(i: number, j: number): boolean {
		return 0 <= i && i < SIZE && 0 <= j && j < SIZE;
	}

	// reveal emptiness across connected component
	function propogateEmpty(i: number, j: number) {
		board[i][j].flagged = false;
		board[i][j].clicked = true;

		for (let dx of [-1, 0, 1]) {
			for (let dy of [-1, 0, 1]) {
				let ii = i + dx;
				let jj = j + dy;

				let shouldPropogate = insideBounds(ii, jj);
				shouldPropogate &&= !board[ii][jj].isBomb;
				shouldPropogate &&= !board[ii][jj].clicked;

				if (shouldPropogate) {
					propogateEmpty(ii, jj);
				}
			}
		}
	}
</script>

<!-- <input type="range" bind:value={SIZE} min={1} max={9} /> -->
<svelte:body on:contextmenu|preventDefault />

<div
	class="board"
	style:grid-template-columns={`repeat(${SIZE}, 1fr)`}
	style:grid-template-rows={`repeat(${SIZE}, 1fr)`}
>
	{#each board as row, i}
		{#each row as cell, j}
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<span
				class="item"
				on:contextmenu={() => onRightClickCell(i, j)}
				on:click={() => onClickCell(i, j)}
				class:flagged={board[i][j].flagged}
				class:clicked={board[i][j].clicked}
			>
				{cell.clicked ? cellContent(i, j) : ''}
			</span>
		{/each}
	{/each}
</div>

<style>
	.board {
		background: #222;
		padding: 2%;
		border-radius: 5%;
		width: 700px;
		height: 700px;

		display: grid;
		grid-gap: 0.5em;
	}

	.item {
		width: 100%;
		height: 100%;
		padding: 2%;
		border-radius: 25%;
		background: #111;
		transition: background-color 0.3s;
	}

	.item:hover {
		background-color: rgb(43, 2, 43);
	}

	.item.flagged {
		background: url('$lib/images/flag2.png') #111;
		image-rendering: crisp-edges;
		background-size: 1.5em;
		background-repeat: no-repeat;
		background-position: center;
	}

	.item.clicked {
		background-color: #333;
	}

	span {
		font-family: 'Minecraft';
		font-weight: black;
		font-size: 2em;
		color: whitesmoke;
		user-select: none;

		display: flex;
		justify-content: center;
		align-items: center;
	}
</style>
