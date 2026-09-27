<script lang="ts">

	import { onMount } from "svelte";
	import { loadPagePromise } from "$lib/store";
	import { letterSlideIn, maskSlideIn } from "$lib/animations";
	import { loadImage, onScrolledIntoView } from "$lib/utils";
    import { scrollAnchorState, viewPortState } from "$lib/state.svelte";

	let section1Element: HTMLElement;
	let section2Element: HTMLElement;
	let profilePicContainer: HTMLElement;

	// Promise which when resolved will trigger svelte animations
	let sectionOneResolve: (value?: any) => void;
	let sectionOnePromise = new Promise((resolve) => sectionOneResolve = resolve);
	let sectionTwoResolve: (value?: any) => void;
	let sectionTwoPromise = new Promise((resolve) => sectionTwoResolve = resolve);

	onMount(async () => {
		// Wait for page to load
		await loadPagePromise;
		// Set navbar about link's y location to top of aboutContainer
		scrollAnchorState.about = section1Element;

		viewPortState.slickscrollInstance.addOffset({
			element: profilePicContainer!,
			speedY: 0.8
		});

		onScrolledIntoView(section1Element, () => sectionOneResolve(true));
		onScrolledIntoView(section2Element, () => sectionTwoResolve(true));
	});

	function titleIn(node: HTMLElement) {
		const titleAnimation = letterSlideIn(node, { delay: 15 });
		titleAnimation.anime();
	}

	// Add parallax scrolling offsets to slickScroll
	function addSlickScrollOffset(node: HTMLElement) {
		viewPortState.slickscrollInstance.addOffset({
			element: node,
			speedY: 0.8
		});
	}

</script>

<div id="content-container" class="about" bind:this={section1Element}>
	{#await sectionOnePromise then _}
		<div class="content-wrapper">
			<h1 class="title" use:titleIn>
				Hey I'm <br>Apurv
			</h1>
			<div in:maskSlideIn={{ duration: 1200, reverse: true, delay: 150 }}>
				<p class="paragraph">
					I'm a web developer from Nasik, Maharashtra. I specialize in designing and developing web experiences<br><br>I work with organizations and individuals to create beautiful, responsive, and scalable web products tailor-made for them. Think we can make something great together? Let's talk over email.
				</p>
			</div>
			<div class="social-button-wrapper">
				<div in:maskSlideIn={{ delay: 400, reverse: true }}>
					<span class="button"><a href="mailto:apuravahire2003@gmail.com" target="_blank" class="clickable sublink link">Email Me</a></span>
				</div>
				<div in:maskSlideIn={{ delay: 700, reverse: true }}>
					<span class="button"><a href="https://github.com/ApurvAhire03" target="_blank" class="clickable sublink link">Github</a></span>
				</div>
			</div>
		</div>
		<div class="profile-image" use:addSlickScrollOffset>
			{#await loadImage("assets/imgs/profile-photo.jpg") then src}
				<img src="{src}" in:maskSlideIn={{ duration: 1200,
					delay: 100,
					reverse: true,
					maskStyles: [
						{ property: "width", value: "100%"},
						{ property: "height", value: "100%"}
					]
				}} alt="Apurv's Profile" class="profile-pic">
			{/await}
		</div>
	{/await}
</div>

<div class="horizontal-flex" bind:this={section2Element}>
	{#await sectionTwoPromise then _}
		<ul class="list first">
			<li class="list-title">
				<div in:letterSlideIn={{ initDelay: 400 }}>
					technical expertise
				</div>
			</li>
			<li>
				<div in:letterSlideIn={{ initDelay: 550 }}>
					Front-end
				</div>
				<div 
					class="flex-item" 
					in:maskSlideIn={{ delay: 600 }}>
					<img src="assets/imgs/svg-icons/svelte.svg" alt="Svelte">
					<img src="assets/imgs/svg-icons/react.svg" alt="React">
				</div>
			</li>
			<li>
				<div in:letterSlideIn={{ initDelay: 650 }}>
					Back-end
				</div>
				<div class="flex-item" in:maskSlideIn={{ delay: 700 }}>
					<img src="assets/imgs/svg-icons/nodejs.svg" alt="node js">
					<img src="assets/imgs/svg-icons/php.svg" alt="php">
				</div>
			</li>
			<li>
				<div in:letterSlideIn={{ initDelay: 750 }}>
					Dev-ops
				</div>
				<div class="flex-item" in:maskSlideIn={{ delay: 800 }}>
					<img src="assets/imgs/svg-icons/firebase.svg" alt="Firebase">
					<img src="assets/imgs/svg-icons/gcp.svg" alt="Google Cloud Platform">
				</div>
			</li>
			<li>
				<div in:letterSlideIn={{ initDelay: 850 }}>
					Mobile
				</div>
				<div class="flex-item" in:maskSlideIn={{ delay: 900 }}>
					<img src="assets/imgs/svg-icons/flutter.svg" alt="flutter">
					<img src="assets/imgs/svg-icons/android.svg" alt="native android">
					<img src="assets/imgs/svg-icons/iOS.svg" alt="native ios">
				</div>
			</li>
		</ul>
	{/await}
</div>


<style lang="sass">

@use "../consts.sass" as consts

@include consts.textStyles()

#content-container.about
	display: flex
	flex-direction: row
	justify-content: space-between
	overflow: hidden
	padding: 0 7vw
	margin-top: 30vh
	position: relative
	padding-bottom: 5vh

	@media only screen and (max-width: 950px)
		flex-direction: column
		margin-top: 14vh
		padding: 0 6vw

	.profile-image
		width: 45%
		overflow: hidden
		margin-top: -20vh
		position: relative

		@media only screen and (max-width: 950px)
			width: 100%
			max-width: 420px
			height: 44vh
			margin: 4vh auto 0
			display: block

		img
			height: 85%
			width: 100%
			border-radius: 12px
			object-fit: cover
			object-position: center 25%
			box-shadow: 0 18px 40px rgba(0, 0, 0, 0.5)

			@media only screen and (max-width: 950px)
				height: 100%

	.content-wrapper
		box-sizing: border-box
		width: 50%
		height: 100%
		margin: 0 2vw
		padding-right: 4vw
		display: flex
		flex-direction: column
		justify-content: center
		margin-top: 5vh
		box-sizing: border-box
		z-index: 2

		@media only screen and (max-width: 950px)
			width: 100%
			margin: 0
			padding-right: 0
			margin-top: 0

		h1
			font-size: clamp(3.2rem, 15vw, 18vh)
			font-weight: 400
			line-height: 85%

		.paragraph
			margin-top: 6vh
			margin-left: 10vw
			position: relative
			width: 75%
			line-height: 170%

			@media only screen and (max-width: 950px)
				width: 100%
				margin-left: 0
				margin-top: 3vh

			&::before
				content: ""
				position: absolute
				height: 1px
				width: 7vw
				right: 110%
				top: 15%
				background-color: white

				@media only screen and (max-width: 950px)
					display: none

		.social-button-wrapper
			font-size: clamp(1.1rem, 2.5vh, 1.8rem)
			margin-left: 10vw
			margin-top: 4vh
			display: inline-flex
			gap: 2vw

			@media only screen and (max-width: 950px)
				margin-left: 0
				margin-top: 3vh
				gap: 1.5rem

			& :global(*:not(:last-child))
				margin-right: 1.5vw

				@media only screen and (max-width: 950px)
					margin-right: 0

.horizontal-flex
	display: flex
	flex-direction: row
	justify-content: space-between
	padding: 0 10vw
	margin-top: 12vh
	width: 100%
	box-sizing: border-box

	@media only screen and (max-width: 950px)
		flex-direction: column
		padding: 0 6vw
		margin-top: 8vh

	.list
		list-style-type: none
		text-align: left
		width: 100%

		@media only screen and (max-width: 950px)
			margin-top: 2vh

		li.list-title
			letter-spacing: 0.4vh
			font-size: clamp(0.8rem, 1.3vh, 1rem)
			font-weight: bold
			color: rgba(255, 255, 255, 0.5)

		li
			font-family: consts.$font
			text-transform: uppercase
			font-size: clamp(0.95rem, 1.8vh, 1.25rem)
			letter-spacing: 0.3vh
			padding: 2.2vh 0
			border-bottom: 1px solid rgba(255, 255, 255, 0.15)
			display: flex
			flex-wrap: wrap
			flex-direction: row
			justify-content: space-between
			align-items: center
			column-gap: 5vw
			row-gap: 1.5vh

			.flex-item
				display: flex
				align-items: center
				gap: 12px

			img
				height: clamp(20px, 2.4vh, 28px)

</style>