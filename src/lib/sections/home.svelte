<script lang="ts">

	import { animate, stagger } from "animejs";
	import { onMount } from "svelte";
	import { loadPagePromise } from "$lib/store";
	import { loadImage } from "$lib/utils";
    import { scrollAnchorState, viewPortState } from "$lib/state.svelte";

	// DOM Node Binds for animations
	let homeContainerElement: HTMLElement = $state()!; // Container
	let backgroundContainerElement: HTMLElement = $state()!;
	let backgroundImageElement: HTMLElement = $state()!; // Offsets

	// Elements for animations
	let titleWord1Element: HTMLElement = $state()!; 
	let titleWord2Element: HTMLElement = $state()!; 
	let shortDetailsElement: HTMLElement = $state()!; 
	let callToActionElement: HTMLElement = $state()!;

	// SVG Paths
	let signaturePath1: SVGPathElement = $state()!; 
	let signaturePath2: SVGPathElement = $state()!;
	let signaturePath3: SVGPathElement = $state()!; 
	let signaturePath4: SVGPathElement = $state()!;

	onMount(async () => {
		await loadPagePromise;
		// Set navbar home link's y location to top of homeContainer
		scrollAnchorState.home = homeContainerElement;

		// Add parallax scrolling offsets to slickScroll
		viewPortState.slickscrollInstance!.addOffset({
			element: backgroundContainerElement,
			speedY: 0.8
		});

		introAnimations();
	})


	// Page load animations
	function introAnimations() {

		const animation = [{ strokeDashoffset: '0' }];

		// Signature animation using svg stroke DashOffset
		signaturePath1.animate(animation, {
			duration: 800,
			delay: 500,
			easing: 'cubic-bezier(.72,.3,.25,1)',
			fill: 'forwards' 
		});
		signaturePath2.animate(animation, {
			duration: 600,
			delay: 1300,
			easing: 'cubic-bezier(.47,.41,.26,1)',
			fill: 'forwards' 
		});
		signaturePath3.animate(animation, {
			duration: 800,
			delay: 1900,
			easing: 'cubic-bezier(.47,.41,.26,1)',
			fill: 'forwards' 
		});
		signaturePath4.animate(animation, {
			duration: 600,
			delay: 2700,
			easing: 'cubic-bezier(.47,.41,.26,1)',
			fill: 'forwards' 
		});


		// Animate background image
		Object.assign(backgroundContainerElement.style, {
			height: "0",
			transform: "scale(1.3)",
		});
		backgroundImageElement.style.transform = "translateY(80%) scale(1.4)";

		animate(backgroundContainerElement, {
			height: "100%",
			scale: 1,
			easing: "cubicBezier(0.165, 0.84, 0.44, 1)",
			duration: 1500,
			delay: 500,
			complete: () => {
				backgroundContainerElement.style.boxShadow = "3px 9px 18px rgba(0, 0, 0, 0.2)";
			}
		});

		animate(backgroundImageElement, {
			translateY: "0",
			scale: 1,
			easing: "cubicBezier(0.165, 0.84, 0.44, 1)",
			duration: 1500,
			delay: 500
		});


		// Animate title elements
		const titleElements = [titleWord1Element, titleWord2Element, shortDetailsElement, callToActionElement];
		titleElements.forEach(e => {
			e.style.transform = "translateY(130%) rotate(10deg)";
		})
		animate(titleElements, {
			rotate: "0",
			translateY: "0%",
			easing: "cubicBezier(0.165, 0.84, 0.44, 1)",
			duration: 900,
			delay: stagger(80, {start: 500})
		});
	}

</script>



<div id="content-container" class="home-hero" bind:this={homeContainerElement}>
	<div class="content-wrapper">
		<div class="flex">
			<div class="flex-wrapper first">

				<svg id="signature" class="h-signature" x="0px" y="0px" viewBox="0 0 240 120">
					<g>
						<path
							bind:this={signaturePath1}
							class="path-1"
							style="fill:none;stroke:#ffffff;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
							d="M 22 75 C 24 64 28 42 38 22 C 43 12 49 13 52 20 C 55 35 55 58 53 78 C 51 86 44 85 40 76 C 35 64 42 51 54 52 C 63 53 71 58 76 63"/>
						<path
							bind:this={signaturePath2}
							class="path-2"
							style="fill:none;stroke:#ffffff;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
							d="M 76 63 C 80 57 84 51 88 50 C 89 53 88 78 87 100 C 86 106 79 104 80 92 C 81 78 84 54 89 52 C 95 50 102 54 101 64 C 100 72 92 76 87 75 C 91 75 96 73 103 67"/>
						<path
							bind:this={signaturePath3}
							class="path-3"
							style="fill:none;stroke:#ffffff;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
							d="M 103 67 C 107 61 111 53 116 52 C 118 56 119 69 124 73 C 128 75 132 60 135 53 C 137 57 138 69 143 73 C 146 75 149 55 154 52 C 158 50 161 53 162 57 C 163 64 165 71 169 74 C 172 75 175 57 179 54 C 181 58 184 71 188 73 C 193 74 198 63 203 53 C 205 49 209 49 212 53"/>
						<path
							bind:this={signaturePath4}
							class="path-4"
							style="fill:none;stroke:#ffffff;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
							d="M 28 92 C 55 95 105 94 155 89 C 182 86 205 81 220 74"/>
					</g>
				</svg>

			</div>
			
			<div class="flex-wrapper second">
				<h1 class = "title">
					<div class="title-mask">
						<div class="word" bind:this={titleWord1Element}>Apurv</div>
					</div><br> 
					<div class="title-mask">
						<div class="word" bind:this={titleWord2Element}>Ahire</div>
					</div>
				</h1>
				<div class="occupation mask">
					<p class = "paragraph" bind:this={shortDetailsElement}>
						web developer from nasik, maharashtra
					</p>
				</div>
				<div class="wrapper action-mask">
					<div class="action" bind:this={callToActionElement}>
						<div class="mask">
							{#await loadImage("assets/imgs/scroll_arrow.png") then src}
								<img src="{src}" alt="">
							{/await}
						</div>
						<div>
							scroll
						</div>
					</div>
				</div>
			</div>

			<div class="parallax-wrapper home-back" bind:this={backgroundContainerElement}>
				{#await loadImage("assets/imgs/home-back.jpg") then src}
					<img src="{src}" bind:this={backgroundImageElement} draggable="false" alt="Home Background" style="width:100%; height: 100%; object-fit: cover;">
				{/await}
			</div>
		</div>
	</div>
</div>



<style lang="sass">

@use "../consts" as consts
@include consts.textStyles()

#content-container.home-hero
	min-height: 100vh
	min-height: 100dvh
	width: 100vw
	padding: 22vh 7vw 10vh
	box-sizing: border-box
	position: relative

	@media only screen and (max-width: 950px)
		padding: 14vh 6vw 6vh
		min-height: 90vh
		min-height: 90dvh

	.content-wrapper
		position: relative
		height: 100%
		box-sizing: border-box
		z-index: 2

	.flex
		z-index: 2
		width: 95%
		height: 100%
		display: flex
		flex-direction: row
		justify-content: space-between
		position: relative
		box-sizing: border-box

		.flex-wrapper
			position: relative
			height: 100%
			display: flex
			flex-direction: column
			justify-content: center

			&.second
				margin-right: 5vw 
				justify-content: flex-end

			h1
				font-weight: 400
				text-shadow: 0px 5px 10px rgba(0, 0, 0, 0.3)
				font-size: clamp(3.2rem, 16vw, 19vh)
				line-height: 85%

			.title-mask
				overflow: hidden
				display: inline-flex

			.word
				font-size: inherit

			.mask
				overflow: hidden

			.h-signature
				width: 35vh
				margin-left: -6vh

			.occupation
				position: relative
				margin-top: 6vh

				@media only screen and (max-width: 950px)
					margin-top: 3vh
					width: 100%

			.action-mask
				margin-top: 8vh
				margin-right: 7vw
				display: inline-flex
				overflow: hidden

				@media only screen and (max-width: 950px)
					margin-top: 4vh

				.action
					font-size: clamp(0.85rem, 2vh, 1.1rem)
					letter-spacing: 0.4vh
					font-family: consts.$font
					text-transform: uppercase
					color: white
					position: relative
					display: inline-flex
					flex-direction: row
					align-items: center

					.mask
						overflow: hidden
						height: 2vh

						img
							height: 2.3vh
							margin-right: 1.5vh
							animation: scrollArrowLoop 3s ease infinite

	.parallax-wrapper
		position: absolute
		left: 0
		z-index: -1
		width: 80%
		height: 100%
		margin-left: 5%
		border-radius: 1.5vh
		overflow: hidden
		box-sizing: border-box
		-webkit-touch-callout: none
		-webkit-user-select: none
		-moz-user-select: none
		-ms-user-select: none
		user-select: none
		transition: box-shadow 0.6s ease
		-webkit-transition: box-shadow 0.6s ease

		@media only screen and (max-width: 1250px)
			&
				opacity: 0.65

		@media only screen and (max-width: 750px)
			&
				opacity: 0.35

		img
			height: 100%
			width: 100%
			object-fit: cover
			border-radius: 1.5vh

@media only screen and (min-width: 1250px)
	.h-signature
		display: block

	.occupation
		width: 100%

	#content-container .flex *
		text-align: left

@media only screen and (max-width: 1250px)
	#content-container .flex *
		text-align: left

	.flex
		justify-content: center !important
		width: 100% !important

		.flex-wrapper 
			&.first
				display: none !important

			&.second
				justify-content: center !important
				margin: 0
				width: 100%

	.parallax-wrapper
		width: 100% !important
		margin-left: 0 !important

@media only screen and (max-width: 750px)
	.occupation
		width: 100% !important


#signature
	.path-1
		stroke-dasharray: 220
		stroke-dashoffset: 220
	
	.path-2
		stroke-dasharray: 195
		stroke-dashoffset: 195

	.path-3
		stroke-dasharray: 235
		stroke-dashoffset: 235

	.path-4
		stroke-dasharray: 205
		stroke-dashoffset: 205


@keyframes scrollArrowLoop
	0%
		transform: translateY(-120%)
	
	30%
		transform: translateY(0%)
	
	70%
		transform: translateY(0%)
	
	100%
		transform: translateY(120%)

</style>