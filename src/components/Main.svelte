<script lang="ts">
	import { flip } from 'svelte/animate';
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';
	import { v4 as uuidv4 } from 'uuid';

	type Position = [number, number];
	type Selection = Position | [];
	type Cell = { value: number; key: string };
	type Gem = { cell: Cell; position: Position };
	type Board = Cell[][];

	let scoringGems: Gem[] = [];

	$: scoringGems && addManaToPlayer();

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

	const checkScoringGems = (currentBoard: Board) => {
		const scoredGems: Gem[] = [];
		currentBoard.forEach((cell, i) =>
			cell.forEach((value, j) => {
				const element = getElementInPosition([i, j], currentBoard) as Cell;
				if (itScores(element, [i, j], currentBoard)) {
					scoredGems.push({ cell: element, position: [i, j] });
				}
			})
		);
		scoringGems = scoredGems;
	};

	function addManaToPlayer() {
		console.log({ scoringGems });
		if (scoringGems.length === 0) {
			return;
		}

		scoringGems.forEach((gem) => {
			console.log(gem);
			const [row, col] = gem.position;
			board[row][col] = { value: 0, key: uuidv4() };
			// Update board by making the upper gems fall down to new position (check each col position for a 0 and replace with upper one. If no upper one, create)
		});
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
					.map(() => ({ value: Math.round(Math.random() * (7 - 1) + 1), key: uuidv4() } as Cell));
			}) as Board;
	};

	let board = randomizePositions();
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
		const currentPosElem = getElementInPosition(currentPos, board);
		const newPosElem = getElementInPosition(newPos, board);
		if (!currentPosElem || !newPosElem) return;
		board[currentPos[0]][currentPos[1]] = newPosElem;
		board[newPos[0]][newPos[1]] = currentPosElem;

		checkScoringGems(board);
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
		board = randomizePositions();
	};

	const handleClickOutside = () => {
		selected = [];
	};

	const getCellTypeClass = (cell: number) => {
		switch (cell) {
			case 0:
				return 'cell';
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
		{#each board as row, i}
			<div class="row">
				{#each row as cell, j (cell.key)}
					<!-- svelte-ignore a11y-click-events-have-key-events -->
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
