<script lang="ts">
	import { flip } from 'svelte/animate';
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';
	import { v4 as uuidv4 } from 'uuid';

	type Position = [number, number];
	type Selection = Position | [];
	type Cell = { value: number; key: number };
	type Board = Cell[][];

	const [send, receive] = crossfade({
		duration: 400,

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	function hasSelection(selected: Position | []): selected is Position {
		return selected.length > 0;
	}

	const itScores = (element: Cell, currentPos: Position, currentBoard: Board) => {
		const scoresLeft =
			element.value ===
				getElementInPosition([currentPos[0], currentPos[1] - 1], currentBoard)?.value &&
			element.value ===
				getElementInPosition([currentPos[0], currentPos[1] - 2], currentBoard)?.value;
		const scoresRight =
			element.value ===
				getElementInPosition([currentPos[0], currentPos[1] + 1], currentBoard)?.value &&
			element.value ===
				getElementInPosition([currentPos[0], currentPos[1] + 2], currentBoard)?.value;
		const scoresUp =
			element.value ===
				getElementInPosition([currentPos[0] - 1, currentPos[1]], currentBoard)?.value &&
			element.value ===
				getElementInPosition([currentPos[0] - 2, currentPos[1]], currentBoard)?.value;
		const scoresDown =
			element.value ===
				getElementInPosition([currentPos[0] + 1, currentPos[1]], currentBoard)?.value &&
			element.value ===
				getElementInPosition([currentPos[0] + 2, currentPos[1]], currentBoard)?.value;
		const scoresCenterVertical =
			element.value ===
				getElementInPosition([currentPos[0] + 1, currentPos[1]], currentBoard)?.value &&
			element.value ===
				getElementInPosition([currentPos[0] - 1, currentPos[1]], currentBoard)?.value;
		const scoresCenterHorizontal =
			element.value ===
				getElementInPosition([currentPos[0], currentPos[1] + 1], currentBoard)?.value &&
			element.value ===
				getElementInPosition([currentPos[0], currentPos[1] - 1], currentBoard)?.value;

		if (scoresUp) {
			console.log('up');
		}
		if (scoresDown) {
			console.log('down');
		}
		if (scoresLeft) {
			console.log('left');
		}
		if (scoresRight) {
			console.log('right');
		}
		if (scoresCenterHorizontal) {
			console.log('center horizontal');
		}
		if (scoresCenterVertical) {
			console.log('center vertical');
		}
		return (
			scoresUp ||
			scoresDown ||
			scoresLeft ||
			scoresRight ||
			scoresCenterHorizontal ||
			scoresCenterVertical
		);
	};

	const randomizePositions = () => {
		return Array(8)
			.fill(null)
			.map(() => {
				return Array(8)
					.fill(null)
					.map(() => ({ value: Math.round(Math.random() * (7 - 1) + 1), key: uuidv4() }));
			});
	};

	let positions = randomizePositions();
	let selected: Selection = [];

	const getElementInPosition = (position: Position, board: Board) => {
		if (board[position[0]] && board[position[1]]) {
			return board[position[0]][position[1]];
		}
		return;
	};

	const areAdjacent = (posA: Selection, posB: Position) => {
		const sameRow = posA[0] === posB[0];
		const sameColumn = posA[1] === posB[1];

		if (sameRow) {
			return posA[1] === posB[1] + 1 || posA[1] === posB[1] - 1;
		}

		if (sameColumn) {
			return posA[0] === posB[0] + 1 || posA[0] === posB[0] - 1;
		}
	};

	const swap = (currentPos: Position, newPos: Position) => {
		const currentPosElem = getElementInPosition(currentPos, positions);
		const newPosElem = getElementInPosition(newPos, positions);
		if (!currentPosElem || !newPosElem) return;
		console.log({ currentPosElem, newPosElem });
		positions[currentPos[0]][currentPos[1]] = newPosElem;
		positions[newPos[0]][newPos[1]] = currentPosElem;

		itScores(currentPosElem, newPos, positions) ? console.log('scored') : console.log('not scored');
		itScores(newPosElem, currentPos, positions) ? console.log('scored') : console.log('not scored');
		selected = [];
	};

	const handleSelect = (e: Event, indices: Position) => {
		e.stopPropagation();
		const adjacent = areAdjacent(selected, indices);
		if (adjacent && hasSelection(selected)) {
			swap(selected, indices);
		} else {
			selected = indices;
		}
	};

	const handleRandomize = () => {
		positions = randomizePositions();
	};

	const handleClickOutside = () => {
		selected = [];
	};

	const getCellTypeClass = (cell: number) => {
		switch (cell) {
			case 1:
				return 'cell cell-water';
			case 2:
				return 'cell cell-fire';
			case 3:
				return 'cell cell-earth';
			case 4:
				return 'cell cell-wind';
			case 5:
				return 'cell cell-skull';
			case 6:
				return 'cell cell-dark';
			case 7:
				return 'cell cell-weapon';
		}
	};
</script>

<svelte:body on:click={handleClickOutside} />

<div class="mainContainer">
	<h1>Puzzle Quest?</h1>
	<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
	<button on:click={handleRandomize}>Random</button>
	<div>
		{#each positions as row, i}
			<div class="row">
				{#each row as cell, j (cell.key)}
					<div
						class={selected[0] === i && selected[1] === j
							? 'cell-container selected'
							: 'cell-container'}
						on:click={(e) => handleSelect(e, [i, j])}
						in:receive={{ key: cell.key }}
						out:send={{ key: cell.key }}
						animate:flip={{ duration: 400 }}
					>
						<div class={getCellTypeClass(cell.value)} />
					</div>
				{/each}
			</div>
		{/each}
	</div>
</div>

<style>
	.mainContainer {
		display: flex;
		flex-direction: column;
		align-items: center;
		border: 1px solid red;
		height: 98vh;
		width: 98vw;
	}
	.row {
		display: flex;
	}
	.cell-container {
		border: 1px solid black;
		padding: 5px;
		cursor: pointer;
	}
	.cell {
		width: 90px;
		height: 90px;
		border-radius: 50%;
	}
	.cell-water {
		background-color: blue;
	}
	.cell-fire {
		background-color: red;
	}
	.cell-earth {
		background-color: green;
	}
	.cell-wind {
		background-color: gold;
	}
	.cell-skull {
		background-color: lightgrey;
	}
	.cell-dark {
		background-color: purple;
	}
	.cell-weapon {
		background-color: gray;
	}
	.selected {
		border-color: red;
		background-color: black;
	}
</style>
