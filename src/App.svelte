<script>
	import P5Canvas from "./P5Canvas.svelte";

	// colors exported from coolers.co
	let colors = [{"name":"Blue Bell","hex":"a09ebb","rgb":[160,158,187],"cmyk":[14,16,0,27],"hsb":[244,16,73],"hsl":[244,18,68],"lab":[66,7,-15]},{"name":"Black Shadows","hex":"baa7b0","rgb":[186,167,176],"cmyk":[0,10,5,27],"hsb":[332,10,73],"hsl":[332,12,69],"lab":[70,8,-2]},{"name":"Illuminating Emerald","hex":"3c896d","rgb":[60,137,109],"cmyk":[56,0,20,46],"hsb":[158,56,54],"hsl":[158,39,39],"lab":[52,-31,8]},{"name":"Independence","hex":"4e4c67","rgb":[78,76,103],"cmyk":[24,26,0,60],"hsb":[244,26,40],"hsl":[244,15,35],"lab":[33,7,-15]},{"name":"Magic Mint","hex":"9df7e5","rgb":[157,247,229],"cmyk":[36,0,7,3],"hsb":[168,36,97],"hsl":[168,85,79],"lab":[91,-31,0]}];
	//let colors = [{"name":"Tea Green","hex":"c8ffbe","rgb":[200,255,190],"cmyk":[22,0,25,0],"hsb":[111,25,100],"hsl":[111,100,87],"lab":[95,-29,26]},{"name":"Indigo Dye","hex":"083d77","rgb":[8,61,119],"cmyk":[93,49,0,53],"hsb":[211,93,47],"hsl":[211,87,25],"lab":[26,8,-38]},{"name":"Spanish Pink","hex":"fcbcb8","rgb":[252,188,184],"cmyk":[0,25,27,1],"hsb":[4,27,99],"hsl":[4,92,85],"lab":[82,22,11]},{"name":"Pastel Pink","hex":"efa7a7","rgb":[239,167,167],"cmyk":[0,30,30,6],"hsb":[0,30,94],"hsl":[0,69,80],"lab":[75,27,11]},{"name":"Maximum Blue Green","hex":"43bccd","rgb":[67,188,205],"cmyk":[67,8,0,20],"hsb":[187,67,80],"hsl":[187,58,53],"lab":[71,-28,-18]}];

	let sketch = function (p5) {
		let windowWidth = window.innerWidth;
		let windowHeight = window.innerHeight;
		let w;
		let columns;
		let rows;
		let board;
		let next;
		let offset = 0; 
		let fr = 2; //framerate

		let getColor = (i, j, neighbors) => {
			let initialColor = colors[0];
			if (neighbors > 2 && neighbors < 5) initialColor = colors[1];
			if (neighbors > 4 && neighbors < 6) initialColor = colors[2];
			if (neighbors > 5 && neighbors < 8) initialColor = colors[3];
			if (neighbors > 7) initialColor = colors[4];
			return `rgba(${initialColor.rgb[0]}, ${initialColor.rgb[1]}, ${initialColor.rgb[2]}, ${(board[i][j].magnitude)/10})`;
			//return `rgba(${((i * j)+adjustedOffset) % 255}, ${(i * (2 * j)) % 255}, ${	(2 * i * j) % 255}, ${i * j * Math.random()})`;
		};
		let getBackgroundColor = () => {
			let color = colors[4];
			return `rgba(${color.rgb[0]}, ${color.rgb[2]}, ${color.rgb[1]}, 0.5)`;
		}
		let getNeighborsNumber = (i, j) => {
			if (board[i][j].alive == 1) {
				let output = 0;
				if (board[i + 1][j].alive == 1) output++;
				if (board[i - 1][j].alive == 1) output++;
				if (board[i][j - 1].alive == 1) output++;
				if (board[i][j + 1].alive == 1) output++;
				if (board[i + 1][j + 1].alive == 1) output++;
				if (board[i - 1][j + 1].alive == 1) output++;
				if (board[i + 1][j - 1].alive == 1) output++;
				if (board[i - 1][j - 1].alive == 1) output++;
				return output;
			} else {
				// if it's an empty slot who cares
				return 0;
			}
		};

		let getLocalMagnitude = (x,y) => {
			let out = 0;
			out += board[x-1][y].magnitude;
			out += board[x+1][y].magnitude;
			out += board[x-1][y+1].magnitude;
			out += board[x-1][y-1].magnitude;
			out += board[x+1][y+1].magnitude;
			out += board[x+1][y-1].magnitude;
			out += board[x][y+1].magnitude;
			out += board[x][y-1].magnitude;
			return out;
		}

		p5.setup = () => {
			p5.frameRate(fr);
			p5.createCanvas(windowWidth, windowHeight);
			w = 25;
			// Calculate columns and rows
			columns = p5.floor(p5.width / w);
			rows = p5.floor(p5.height / w);
			// Wacky way to make a 2D array is JS
			board = new Array(columns);
			for (let i = 0; i < columns; i++) {
				board[i] = new Array(rows);
			}
			// Going to use multiple 2D arrays and swap them
			next = new Array(columns);
			for (let i = 0; i < columns; i++) {
				next[i] = new Array(rows);
			}
			p5.init();
		};

		p5.draw = () => {
			p5.background(getBackgroundColor());
			p5.generate();
			for (let i = 0; i < columns; i++) {
				offset = (offset + 1) % (p5.width / w);
				for (let j = 0; j < rows; j++) {
					if (board[i][j].alive == 1) {
						let neighbors = getNeighborsNumber(i,j); // between 0 and 8
						p5.fill(getColor(i, j, neighbors));
						p5.strokeWeight(0);
						p5.circle((i * w)+offset, j * w, 
					 		((board[i][j].magnitude + 1) * 10)
						);
					}
				}
			}
		};

		// reset board when mouse is pressed
		p5.mousePressed = () => {
			p5.init();
		};

		// Fill board randomly
		p5.init = () => {
			for (let i = 0; i < columns; i++) {
				for (let j = 0; j < rows; j++) {
					// Lining the edges with 0s
					if (i == 0 || j == 0 || i == columns - 1 || j == rows - 1)
						board[i][j] = {
							alive: 0,
							magnitude: 0,
						};
					// Filling the rest randomly
					else
						board[i][j] = {
							alive: p5.floor(p5.random(2)),
							magnitude: p5.floor(p5.random(10)),
						};
					next[i][j] = {
						alive: 0,
						magnitude: 0,
					};
				}
			}
			
		};

		// The process of creating the new generation
		p5.generate = () => {
			// Loop through every spot in our 2D array and check spots neighbors
			for (let x = 1; x < columns - 1; x++) {
				for (let y = 1; y < rows - 1; y++) {
					// Add up all the states in a 3x3 surrounding grid
					let neighbors = getNeighborsNumber(x, y);
					let localMagnitude = getLocalMagnitude(x, y);
					// Rules of Life
					if (board[x][y].alive == 1) {
						next[x][y] = {
							alive: (p5.random(11) < (3 + neighbors)) ? 1 : 0, //(p5.floor(p5.random(0,10)) > 3) ? 1 : 0, // chance to kill if alive
							magnitude: neighbors+p5.random(0, 3),
						};
					} else if (board[x][y].alive == 0 && neighbors >= 2 && ((p5.random(10) + localMagnitude) > 20)) {
						next[x][y] = {
							alive: 1,
							magnitude: localMagnitude/8 + p5.random(0, 3),
						};
					} else if (board[x][y].alive == 0 && ((p5.random(30) + localMagnitude > 55) || (p5.random(30) > 28))  ) {
						next[x][y] = {
							alive: 1,
							magnitude: 1,
						};
					} else {
						next[x][y] = board[x][y]; // Stasis
					}
				}
			}

			// Swap!
			let temp = board;
			board = next;
			next = temp;
		};
	};
</script>

<main>
	<P5Canvas {sketch} />
</main>

<style>
	main {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
