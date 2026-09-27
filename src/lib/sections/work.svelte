<script lang="ts">

	import { getGPUTier } from 'detect-gpu';
	import { onMount } from "svelte";
	import { fade } from "svelte/transition";
	import { ImageRenderer } from "$lib/effects/work-slider/renderer";
	import { letterSlideIn, letterSlideOut, maskSlideIn, maskSlideOut, workImageIntro, workListIntro } from "$lib/animations";
	import { loadPagePromise } from "$lib/store";
	import { dataState, scrollAnchorState, viewPortState, workScrollState } from "$lib/state.svelte";
	import { loadImage, onScrolledIntoView } from "$lib/utils";


	let workContainer: HTMLElement;
	let container: HTMLElement; 
	let listContainer: HTMLElement; // Containers for Three meshes
	let images: HTMLImageElement[] = []; // Array of images to be passed to WebGL Shader
	let workItems: HTMLElement[] = []; // Array of workItems to be animated

	let breakTitleWords: boolean = false;
	let currentActive: number = -1; // The work item to expand

	let inViewResolve: (_: boolean) => void;
	const inViewPromise: Promise<boolean> = new Promise((resolve) => {
		inViewResolve = resolve;
	});


	// Slider calculations and rendering
	class WorkSlider {

		currentMouseX = 0; 
		initialMouseX = 0;
		currentPosition = 0; 
		targetPosition = 0; 
		initialPosition = 0;
		offsetSpeed = 5000; 
		lerpSpeed = 0.1;

		onHold = (e: MouseEvent) => {
			e.preventDefault();
			if (currentActive >= 0 || workScrollState.active || (e.target! as HTMLElement).classList.contains("button")) return;

			this.initialMouseX = e.clientX;
			this.currentMouseX = e.clientX;
			workScrollState.active = true;

			if (workScrollState.active) {
				let style = window.getComputedStyle(listContainer);
				let matrix = new WebKitCSSMatrix(style.transform);

				this.initialPosition = matrix.m41;
			}
		}

		onRelease() {
			workScrollState.active = false;
		}
	
		onMouseMove = (e: MouseEvent) => {
			e.preventDefault();
			if (!workScrollState.active) return; 
			this.currentMouseX = e.clientX;

			let diff = (this.currentMouseX - this.initialMouseX) * -1;
			this.targetPosition = Math.round((this.initialPosition - (this.offsetSpeed * (diff / document.body.clientWidth))) * 100) / 100;
		}

		animate = () => {
			if (currentActive < 0) {
				let endPoint = listContainer.offsetWidth - document.body.clientWidth
				if (endPoint < 0) endPoint = listContainer.offsetWidth;

				// Checks for disabling over-scrolling
				if (this.targetPosition > 0) this.targetPosition = 0;
				if (this.targetPosition <= (endPoint * -1)) this.targetPosition = - endPoint;
			}

			// Lerp easing
			this.currentPosition = this.lerp(this.currentPosition, this.targetPosition, this.lerpSpeed);
			
			workScrollState.speed = Math.round((this.currentPosition - this.targetPosition) * 100) / 100; // Set Svelte Store value for the Canvas effect
			listContainer.style.transform = `translate3d(${ Math.round(this.currentPosition * 100) / 100 }px, 0px, 0px)`;

			requestAnimationFrame(() => this.animate());
		}

		lerp(start: number, end: number, t: number) {
			return start * (1 - t) + end * t;
		}
	}


	
	const slider = new WorkSlider();

	onMount(async () => {

		onScrolledIntoView(workContainer, () => inViewResolve(true));

		// GPU Tier to decide if effects should be enabled
		const gpuTier = await getGPUTier();
		// Svelte store for checking if device is a mobile device
		viewPortState.isMobile = gpuTier.isMobile!;

		await loadPagePromise;
		scrollAnchorState.work = workContainer;

		listContainer.style.transform = "translate3d(0px, 0px, 0px)";

		// ThreeJS warping effect if device can handle it
		if (gpuTier.tier >= 2 && !gpuTier.isMobile && gpuTier.fps! >= 30) new ImageRenderer(container, images);
	});

	// Move slider to active item when it is active
	function toggleActiveItem(i: number) {
		currentActive = (currentActive == i) ? -1 : i;
		if (currentActive >= 0) slider.targetPosition = -(workItems[i].offsetLeft - (window.innerWidth / 4) + window.innerWidth / 10);
	}

	function titleSlide(node: HTMLElement) {
		let title = letterSlideIn(node, { delay: 5, breakWord: false });
		title.anime({
			onComplete: () => breakTitleWords=true
		});
	}

</script>




<div id="content-container" class="work-click-area" bind:this="{ workContainer }">

	<div class="content-wrapper" 
		role="listbox"
		tabindex="0"
		onmousedown={slider.onHold}
		onmouseup={slider.onRelease}
		onmouseleave={slider.onRelease}
		onmousemove={slider.onMouseMove}
		bind:this={container}
		class:disabled={currentActive >= 0}
		use:workListIntro={{ promise: inViewPromise, onComplete: async () => {
			 // Begin slider animations and effects once slider is animated in and if device is not a phone
			const gpuTier = await getGPUTier();
			if (!gpuTier.isMobile) slider.animate();
		} }}
	>
		<div class:mobile={viewPortState.isMobile}>
			<ul class="work-list" 
				bind:this={listContainer} 
				class:hold={workScrollState.active}>

				<!-- Work items -->
				{#each dataState.workData! as item, i}
					<li use:workImageIntro={{ promise: inViewPromise, delay: i*30 }}>
						<div class="list-item clickable passive" 
							class:ambient="{ currentActive !== i && currentActive >= 0 }" 
							class:active="{ currentActive === i }" 
							bind:this={ workItems[i] }>

							<div class="img-wrapper">
								{#await loadImage(`assets/imgs/work-back/${item.id}/cover.jpg`) then src}
									<img bind:this={images[i]} src="{src}" ondragstart={(e) => {e.preventDefault()}} draggable="false" alt="{item.title} Background">
								{/await}
							</div>
							{#await inViewPromise then _}
								<div class="text-top-wrapper" class:hidden={currentActive >= 0 || workScrollState.active}>
									<p 
										class="item-index"
										in:maskSlideIn={{
											delay: (i*30)+100,
											reverse: true
										}}>
										{(i.toString().length > 1) ? (i+1) : "0"+(i+1).toString()}
									</p>
								</div>
								
								<div class="text-wrapper" class:hidden={currentActive >= 0 || workScrollState.active}>
									<h1 
										class="item-title" 
										>
										<span in:maskSlideIn={{
											duration: 900, 
											delay: (i*30)+300,
											reverse: true 
										}}>
											{item.title}
										</span>
									</h1>

									<button 
										class="button item-link interactive"
										onclick={() => toggleActiveItem(i)}
										in:maskSlideIn={{
											duration: 900,
											delay: (i*30)+450,
											reverse: true
										}}>
										view
									</button>
								</div>
							{/await}
						</div>
					</li>
				{/each}
			</ul>
		</div>

		<!-- Active work item details (When a work item is clicked) -->
		{#if currentActive !== -1}
			<div class="details-container">
				<div class="wrapper">
					<div class="top-align">
						<div class="wrapper">
							<div class="index">
								<div in:maskSlideIn={{ reverse: true }} out:maskSlideOut>
									{#if (currentActive < 9)}
										{"0"+(currentActive+1)}
									{:else}
										{currentActive+1}
									{/if}
								</div>
							</div>
							<span class="line" transition:fade></span>
							<h6 class="caption">
								<div in:maskSlideIn={{ reverse: true }} out:maskSlideOut>{dataState.workData![currentActive].details.summary}</div>
							</h6>
						</div>
					</div>
					
					<div class="mid-align">
						<h1 class="title" 
							use:titleSlide
							out:letterSlideOut
							class:breakTitleWords
							onintroend={() => setTimeout(() => breakTitleWords = true, 100)}
							onoutrostart={() => setTimeout(() => breakTitleWords = false, 100)}>

							{dataState.workData![currentActive].title}
						</h1>
						<button class="close-button-wrapper interactive" onclick={() => toggleActiveItem(currentActive)}>
							<div 
								class ="close-button"
								in:maskSlideIn={{ reverse: true }} 
								out:maskSlideOut>

								&times;
							</div>
						</button>
					</div>
					
					<div class="bottom-align">
						<div>
							<div in:maskSlideIn={{ reverse: true }} out:maskSlideOut>
								<p class="paragraph">
									{dataState.workData![currentActive].details.description}
								</p>
							</div>
						</div>
						<div in:maskSlideIn={{ reverse: true }} out:maskSlideOut>
							<div class="links">
								{#each dataState.workData![currentActive].links as link}
									<a href={link.link} target="_blank" class="button">{link.text}</a>
								{/each}
							</div>
						</div>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>


<style lang="sass">

@use "../consts" as consts
@include consts.textStyles()

#content-container.work-click-area
	margin-top: 30vh

	@media only screen and (max-width: 950px)
		margin-top: 14vh

#content-container.work-click-area .content-wrapper
	display: flex
	flex-direction: column
	cursor: grab
	position: relative

	&.disabled
		cursor: default !important

		.mobile ul.work-list
			opacity: 0

	.mobile
		width: 100%
		height: 100%
		overflow-x: auto
		-webkit-overflow-scrolling: touch
		scroll-snap-type: x mandatory
		scrollbar-width: none

		&::-webkit-scrollbar
			display: none
	
	*
		-webkit-touch-callout: none
		-webkit-user-select: none
		-moz-user-select: none
		-ms-user-select: none
		user-select: none

	.details-container
		position: absolute
		left: 0
		top: 0
		height: 100%
		width: 100%
		display: flex
		flex-direction: row
		justify-content: space-between
		box-sizing: border-box
		padding: 0 14vw
		z-index: 10

		@media only screen and (max-width: 950px)
			padding: 0 6vw

		.wrapper
			text-align: left
			position: relative
			display: flex
			flex-direction: column
			justify-content: space-between
			flex-basis: 100%

			.top-align
				.wrapper
					display: inline-flex
					flex-direction: row
					align-items: center
					justify-content: left
					position: relative

					h6.caption
						position: relative
						font-family: consts.$font
						text-transform: uppercase
						font-weight: normal
						font-size: clamp(0.9rem, 1.5vw, 1.9vh)

					.index
						font-family: consts.$font
						position: relative
						font-size: clamp(1rem, 1.8vw, 2.1vh)

					span.line
						width: 300%
						margin: 0 10%
						height: 1.5px
						background-color: white

			.mid-align	
				display: flex
				flex-direction: row
				align-items: center
				justify-content: space-between

				h1.title
					position: relative
					font-family: consts.$titleFont
					font-size: clamp(2.5rem, 7vw, 6rem)
					text-transform: lowercase
					font-weight: normal
					word-wrap: break-word
					white-space: normal
					line-height: 100%

					&.breakTitleWords
						display: inline-block
						max-width: min-content

				.close-button
					cursor: pointer
					font-size: clamp(2rem, 3.3vw, 3rem)

			@media only screen and (max-width: 750px)
				.mid-align
					flex-direction: column
					justify-content: flex-start
					align-items: flex-start

					h1.title
						font-size: clamp(2.2rem, 11vw, 3.8rem)

					.close-button-wrapper
						position: absolute
						top: 0
						right: 0

						.close-button
							font-size: 4vh

			
			.bottom-align
				display: flex
				flex-direction: row
				justify-content: space-between
				align-items: center
				gap: 5vh

				*
					flex-grow: 1
					flex-basis: 0

				p
					font-size: clamp(0.9rem, 1.2vw, 1.3vh)
					width: 65%
					line-height: 160%

				.links
					position: relative
					display: flex
					flex-direction: column
					align-items: flex-end
					gap: 2vh

					.button
						font-size: clamp(0.95rem, 1.1vw, 1.4rem)
						letter-spacing: 0.2vw
						text-transform: uppercase
						text-decoration: none
						line-height: 160%

					.underline
						display: none

			@media only screen and (max-width: 750px)
				.bottom-align
					flex-direction: column
					justify-content: flex-start
					align-items: flex-start
					gap: 1.5vh

					p
						font-size: clamp(0.95rem, 3.8vw, 1.1rem) !important
						width: 100%

					.links
						margin: 1.5vh 0
						align-items: flex-start
						flex-direction: row
						flex-wrap: wrap
						gap: 1.5rem
						
						.button
							font-size: 1.1rem
							position: relative

						.underline
							display: inline-block
							position: absolute
							width: 100%
							height: 2px
							bottom: 0
							left: 0
							background-color: white

						div.line
							display: none

	ul.work-list
		margin-top: auto
		margin: auto 0
		padding: 0 5vw
		list-style-type: none
		display: flex
		flex-direction: row
		align-items: center
		height: 75vh
		min-width: min-content
		opacity: 1
		transition: opacity 0.5s ease
		-webkit-transition: opacity 0.5s ease

		&.hold
			.list-item
				height: 42vh !important
				width: 38vw !important

		.list-item
			display: inline-flex
			justify-content: flex-end
			overflow: hidden
			height: 48vh
			width: 42vw
			box-sizing: border-box
			position: relative
			z-index: 3
			margin-right: 8vw
			transition: width 0.7s cubic-bezier(0.25, 1, 0.5, 1), height 0.7s cubic-bezier(0.25, 1, 0.5, 1), margin 0.8s cubic-bezier(0.25, 1, 0.5, 1)

			*
				transition: opacity 0.3s ease
				-webkit-transition: opacity 0.3s ease

			&.active
				height: 56vh
				width: 58vw
				margin-right: 14vw
				margin-left: 6vw

				.img-wrapper
					width: 100%
					margin-right: 0

			&.ambient
				height: 40vh
				width: 36vw

			.hidden
				opacity: 0

			.img-wrapper
				overflow: hidden
				height: 100%
				z-index: 1
				position: relative
				width: 88%
				margin-right: 12%
				border-radius: 8px
				box-shadow: 0 16px 36px rgba(0, 0, 0, 0.45)
				
				img
					height: 100%
					width: 100%
					object-fit: cover
					object-position: top center
					position: absolute
					top: 0
					left: 0
					opacity: 0.65

			.text-top-wrapper
				position: absolute
				top: 4vh
				right: 0
				z-index: 2
				word-wrap: break-word
				white-space: normal
				text-align: right

				.item-index
					font-size: 1vw
					letter-spacing: 0.1vw
					text-transform: uppercase
					font-family: consts.$font

			.text-wrapper
				box-sizing: border-box
				display: flex
				flex-direction: column
				justify-content: flex-end
				position: absolute
				bottom: 4vh
				right: 0
				max-width: 55%
				text-align: right
				z-index: 2

				.button
					font-size: 1.1vw
					letter-spacing: 0.1vw
					margin-top: 1.5vh
					text-transform: uppercase

				.item-title
					font-family: consts.$font
					font-weight: normal
					font-size: 2.2vw
					z-index: 0
					opacity: 1
					letter-spacing: 0.05vw
					line-height: 110%
					text-transform: lowercase
					word-wrap: break-word
					white-space: normal
					text-shadow: 0px 4px 12px rgba(0, 0, 0, 0.8)


				.inline-wrapper
					width: 100%
					position: relative
					margin-top: 1vh

					*
						display: inline-block

		@media only screen and (max-width: 1450px)
			.list-item
				width: 48vw
				height: 46vh

				.text-wrapper
					.item-title
						font-size: 2.8vw

					.button
						font-size: 1.3vw

		@media only screen and (max-width: 1110px)
			.list-item
				width: 62vw
				height: 44vh
				margin-right: 8vw

				.text-top-wrapper
					.item-index
						font-size: 1.8vh

				.text-wrapper
					max-width: 65%

					.item-title
						font-size: 4vw

					.button
						font-size: 1.8vh

		@media only screen and (max-width: 650px)
			.list-item
				width: 84vw
				height: 38vh
				margin-right: 6vw

				.text-wrapper
					max-width: 75%
					bottom: 3vh

					.item-title
						font-size: 3.2vh

</style>