<script>
	import { onMount } from 'svelte';
	import Two from 'two.js';
	import Matter from 'matter-js';

	let container;

	function sleep(ms) {
		return new Promise((resolve) => setTimeout(resolve, ms));
	}

	onMount(() => {
		var r = document.querySelector(':root');
		var text = document.querySelector('#text');

		const two = new Two({
			type: Two.Types.canvas,
			fullscreen: true,
			autostart: true
		}).appendTo(container);

		const engine = Matter.Engine.create();
		engine.world.gravity.y = 0.01;

		const render = Matter.Render.create({
			element: document.body,
			engine: engine,
			options: {
				width: window.innerWidth,
				height: window.innerHeight,
				wireframe: false
			}
		});

		Matter.Render.run(render);

		const runner = Matter.Runner.create();
		Matter.Runner.run(runner, engine);

		const bounds = {
			length: 5000,
			thickness: 5000,
			properties: {
				isStatic: true
			}
		};

		bounds.left = createBoundary(two, bounds.thickness, bounds.length);
		bounds.right = createBoundary(two, bounds.thickness, bounds.length);
		bounds.bottom = createBoundary(two, bounds.length, bounds.thickness);

		
		//Matter.World.add(engine.world, [bounds.left.entity, bounds.right.entity]);
		Matter.World.add(engine.world,[]);

		const entities = [];
		const mouse = addMouseInteraction(engine);

		two
			.bind('update', (frameCount, timeDelta) =>
				update(frameCount, timeDelta, engine, entities, mouse)
			);

		resize(two, bounds);

		async function run_website() {
			const texts = [
				'Hello World',
				"I'm",
				'I do stuff',
				'like',
				'I know some things too',
				'including but not limited to:',
				'python',
				'lua',
				'js',
				'svelte',
				'html',
				'css'
			];
			for (const word of texts) {
				for (let i = 0; i < word.length; i++) {
					text.innerHTML = text.innerHTML + word[i];
					await sleep(200);
				}
				var index = texts.indexOf(word);
				if (index == 1) {
					addLetters(two, entities, engine);
				}
				if (index == 3) {
					add_know(two, entities, engine);
				}
				await sleep(2000);
				console.log(text.innerHTML.length, text.innerHTML);
				while (text.innerHTML.length > 0) {
					text.innerHTML = text.innerHTML.slice(0, -1);
					await sleep(100);
				}
				await sleep(500);
			}
		}

		run_website();
	});

	function createBoundary(two, width, height) {
		const rectangle = two.makeRectangle(0, 0, width, height);
		rectangle.visible = false;
		rectangle.entity = Matter.Bodies.rectangle(0, 0, width, height, {
			isStatic: true
		});
		rectangle.entity.position = rectangle.position;
		return rectangle;
	}

	function addMouseInteraction(engine) {
		const mouse = Matter.Mouse.create(document.body);
		const mouseConstraint = Matter.MouseConstraint.create(engine, {
			mouse: mouse,
			constraint: {
				stiffness: 0.1,
				render: {
					visible: false // Make the mouse constraint visible for debugging
				}
			}
		});

		Matter.Events.on(mouseConstraint, 'mousedown', (event) => {
			const mousePosition = event.mouse.position;
			const bodies = Matter.Composite.allBodies(engine.world);
			const body = Matter.Query.point(bodies, mousePosition)[0];

			if (body) {
				body.isStatic = false;
				body.restitution = 0.8; // Add some bounciness
			}
		});

		Matter.Events.on(mouseConstraint, 'enddrag', (event) => {
			const body = event.body;
			if (body) {
				body.isStatic = false;
			}
		});

		Matter.World.add(engine.world, mouseConstraint);
		return mouseConstraint;
	}

	function resize(two, bounds) {
		const { length, thickness } = bounds;
		const vector = new Two.Vector();
		vector.x = -thickness / 2;
		vector.y = two.height / 2;
		Matter.Body.setPosition(bounds.left.entity, vector);

		vector.x = two.width + thickness / 2;
		vector.y = two.height / 2;
		Matter.Body.setPosition(bounds.right.entity, vector);

		vector.x = two.width / 2;
		vector.y = two.height + thickness / 2;
		Matter.Body.setPosition(bounds.bottom.entity, vector);
	}

	function addLetters(two, entities, engine) {
		const letters = ['H', 'E', 'R', 'A'];

		const defaultStyles = {
			size: two.width * 0.28,
			weight: 700,
			fill: '#FF8F8F',
			family: 'Montserrat, Arial, sans-serif',
			alignment: 'center',
			margin: {
				top: 0,
				left: 0,
				right: 0,
				bottom: 0
			}
		};

		const xOffset = two.width / (letters.length + 1);

		letters.forEach((letter, index) => {
			const group = new Two.Group();
			const text = new Two.Text(letter, 0, 0, defaultStyles);
			group.isWord = true;

			const ox = (index + 1) * xOffset;
			const oy = -100; // Start above the screen

			group.translation.set(ox, oy);
			text.translation.y = 14;

			const rect = text.getBoundingClientRect();
			const rectangle = new Two.Rectangle(0, 0, rect.width, rect.height);
			rectangle.fill = 'rgba(255, 0, 0, 0.3)';
			rectangle.noStroke();
			rectangle.visible = false;

			const entity = Matter.Bodies.rectangle(ox, oy, rect.width, rect.height, {
				restitution: 0.8 // Add bounciness
			});
			entity.scale = new Two.Vector(rect.width, rect.height);
			entity.object = group;
			entities.push(entity);

			group.text = text;
			group.rectangle = rectangle;
			group.entity = entity;

			group.add(rectangle, text);
			two.add(group);
		});

		Matter.World.add(engine.world, entities);
	}

	function add_know(two, entities, engine) {
		const letters = ['tatom', 'blockcoin', 'this website', 'roblox games'];

		const defaultStyles = {
			size: two.width * 0.07,
			weight: 700,
			fill: '#FF8F8F',
			family: 'Montserrat, Arial, sans-serif',
			alignment: 'center',
			margin: {
				top: 0,
				left: 0,
				right: 0,
				bottom: 0
			}
		};

		const xOffset = two.width / (letters.length + 1);

		letters.forEach((letter, index) => {
			const group = new Two.Group();
			const text = new Two.Text(letter, 0, 0, defaultStyles);
			group.isWord = true;

			const ox = (index + 1) * xOffset;
			const oy = -100; // Start above the screen

			group.translation.set(ox, oy);
			text.translation.y = 14;

			const rect = text.getBoundingClientRect();
			const rectangle = new Two.Rectangle(0, 0, rect.width, rect.height);
			rectangle.fill = 'rgba(255, 0, 0, 0.3)';
			rectangle.noStroke();
			rectangle.visible = false;

			const entity = Matter.Bodies.rectangle(ox, oy, rect.width, rect.height, {
				restitution: 0.8 // Add bounciness
			});
			entity.scale = new Two.Vector(rect.width, rect.height);
			entity.object = group;
			entities.push(entity);

			group.text = text;
			group.rectangle = rectangle;
			group.entity = entity;

			group.add(rectangle, text);
			two.add(group);
		});

		Matter.World.add(engine.world, entities);
	}

	function update(frameCount, timeDelta, engine, entities, mouse) {
		const allBodies = Matter.Composite.allBodies(engine.world);
		Matter.MouseConstraint.update(mouse, allBodies);
		Matter.MouseConstraint._triggerEvents(mouse);
		Matter.Engine.update(engine);

		for (const entity of entities) {
			entity.object.position.copy(entity.position);
			entity.object.rotation = entity.angle;
		}
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div id="bg" />
<div class="container">
	<div id="text"></div>
</div>

<div bind:this={container} style="width: 100%; height: 100%;"></div>

<style>
	#bg {
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		position: fixed;
		background-color: var(--back);
	}

	#matter-js {
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		position: fixed;
		z-index: 100;
	}

	:root {
		--front: #ff8f8f;
		--back: #ffe5e5;
	}

	.container {
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		position: fixed;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	#text {
		font-family: Montserrat;
		color: var(--front);
		font-weight: 700;
		font-size: 58px;
	}
</style>
