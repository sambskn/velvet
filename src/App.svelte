<script>
import { sign } from "crypto";

	import P5Canvas from "./P5Canvas.svelte";

	// colors exported from coolers.co
	let colors = [{"name":"Royal Blue Dark","hex":"0a1045","rgb":[10,16,69],"cmyk":[86,77,0,73],"hsb":[234,86,27],"hsl":[234,75,15],"lab":[8,19,-34]},{"name":"Dark Turquoise","hex":"00c2d1","rgb":[0,194,209],"cmyk":[100,7,0,18],"hsb":[184,100,82],"hsl":[184,100,41],"lab":[72,-35,-19]},{"name":"Middle Yellow","hex":"f9e900","rgb":[249,233,0],"cmyk":[0,6,100,2],"hsb":[56,100,98],"hsl":[56,100,49],"lab":[91,-13,90]},{"name":"Rajah","hex":"f6af65","rgb":[246,175,101],"cmyk":[0,29,59,4],"hsb":[31,59,96],"hsl":[31,89,68],"lab":[77,18,48]},{"name":"Shocking Pink","hex":"ed33b9","rgb":[237,51,185],"cmyk":[0,78,22,7],"hsb":[317,78,93],"hsl":[317,84,56],"lab":[56,79,-28]}]
	//let colors = [{"name":"Tea Green","hex":"c8ffbe","rgb":[200,255,190],"cmyk":[22,0,25,0],"hsb":[111,25,100],"hsl":[111,100,87],"lab":[95,-29,26]},{"name":"Indigo Dye","hex":"083d77","rgb":[8,61,119],"cmyk":[93,49,0,53],"hsb":[211,93,47],"hsl":[211,87,25],"lab":[26,8,-38]},{"name":"Spanish Pink","hex":"fcbcb8","rgb":[252,188,184],"cmyk":[0,25,27,1],"hsb":[4,27,99],"hsl":[4,92,85],"lab":[82,22,11]},{"name":"Pastel Pink","hex":"efa7a7","rgb":[239,167,167],"cmyk":[0,30,30,6],"hsb":[0,30,94],"hsl":[0,69,80],"lab":[75,27,11]},{"name":"Maximum Blue Green","hex":"43bccd","rgb":[67,188,205],"cmyk":[67,8,0,20],"hsb":[187,67,80],"hsl":[187,58,53],"lab":[71,-28,-18]}];

	let sketch = function (p5) {
		let windowWidth = window.innerWidth;
		let windowHeight = window.innerHeight;
		let offset = 0; 
		let fr = 12
		; //framerate
		let magnitude = windowWidth * 0.5;


		p5.setup = () => {
			p5.frameRate(fr);
			p5.createCanvas(windowWidth, windowHeight);
			p5.init();
		};

		p5.draw = () => {
			p5.background(colors[0].rgb);
			p5.generate();
			if (p5.frameCount % 480) offset ++;
			let radius = 100;
			let centerX = radius*p5.cos(offset/16) + (windowWidth/2);
			centerX = (centerX + p5.mouseX)/2;
			let centerY = radius*p5.sin(offset/16) + (windowHeight/2);
			centerY = (centerY + p5.mouseY)/2;
			let slopeDiff = ((windowWidth/2) - centerX) * (10/25);
			
			let rowHeight = 20;
			for (let i = -100; i < windowHeight +100; i++) {
				if (i < centerY) {
					p5.strokeWeight(1);
					p5.stroke(colors[2].rgb);
					if ((i + offset)%(rowHeight) == 0) {
						p5.line(0, i-slopeDiff, windowWidth, i+slopeDiff);
					}
				} else {
					p5.strokeWeight(1);
					p5.stroke(colors[1].rgb);
					if ((i - offset)%(rowHeight+(offset % 10)) == 0) {
						p5.line(0, i-slopeDiff, windowWidth, i+slopeDiff);
					}
					
					
				}
			}

			let mouseXBuffer = 75;
			p5.stroke(colors[3].rgb);
			p5.fill(`rgba(${colors[3].rgb[0]},${colors[3].rgb[1]},${colors[3].rgb[2]},0.25)`);
			for (let i = -100; i < windowWidth + 100; i++) {
				if (i < (centerX - mouseXBuffer)) {
					if ((i+offset) % p5.floor(mouseXBuffer + 0.1*p5.abs(i- centerX)) == 0) {
						p5.rect(
							0, 0,
							i, centerY + mouseXBuffer + 0.33*p5.abs(i-(centerX-slopeDiff))
						)
					}
				} else if (i > centerX + mouseXBuffer) {
					if ((i-offset) % p5.floor(mouseXBuffer + 0.1*p5.abs(i- centerX)) == 0) {
						p5.rect(
							i, 0,
							windowWidth, centerY + mouseXBuffer + 0.33*p5.abs(i-(centerX-slopeDiff)),
						)
					}
				} else {

				}
			}
			
			
			

			
			
			
		};

		// reset board when mouse is pressed
		p5.mousePressed = () => {
			p5.init();
		};

		
		p5.init = () => {
		};

		// The process of creating the new generation
		p5.generate = () => {
			
		};
	};
</script>

<main>
	<h1>sup dude</h1>
	<P5Canvas {sketch} />
</main>

<style>
	main {
		display: flex;
		align-items: center;
		justify-content: center;
	}
	h1 {
		position: absolute;
		top: 1rem;
		left: 2rem;
	}
</style>
