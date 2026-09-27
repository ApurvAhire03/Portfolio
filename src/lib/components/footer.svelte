<script lang="ts">

    import { onMount } from "svelte";
    import { letterSlideIn, maskSlideIn } from "$lib/animations";
    import { loadPagePromise } from "$lib/store";
    import { onScrolledIntoView } from "$lib/utils";
    import { dataState } from "$lib/state.svelte";

    let footerContainerElement: HTMLElement = $state()!
    let logoElement: HTMLElement = $state()!; 
    let creditsElement: HTMLElement = $state()!; 
    let statusElement: HTMLElement = $state()!; 
    let fullEmailLinkElement: HTMLElement = $state()!;

    let signaturePath1: SVGPathElement = $state()!; 
    let signaturePath2: SVGPathElement = $state()!; 
    let signaturePath3: SVGPathElement = $state()!;
    let signaturePath4: SVGPathElement = $state()!; 

    const currentYear = new Date().getFullYear();
    
    function introAnimations() {

        // Scroll activated animations powered by anime instead of svelte transitions
        const logoAnimate = maskSlideIn(logoElement, {});
        const fullEmailLinkAnimate = letterSlideIn(fullEmailLinkElement, { delay: 6, initDelay: 150 });
        const creditsAnimate = maskSlideIn(creditsElement, { delay: 150 });
        const statusAnimate = letterSlideIn(statusElement, { delay: 6, initDelay: 100 });

        // Intersection observer to run animations when footer is in scroll view
        onScrolledIntoView(footerContainerElement, () => {
            logoAnimate.anime();
            creditsAnimate.anime();
            fullEmailLinkAnimate.anime();
            statusAnimate.anime();

            // Signature SVG animation
            let animation = [{ strokeDashoffset: '0' }];

            // Signature animation using svg strokDashOffset
            signaturePath1.animate(animation, {
                duration: 800,
                delay: 0,
                easing: 'cubic-bezier(.72,.3,.25,1)',
                fill: 'forwards' 
            });
            signaturePath2.animate(animation, {
                duration: 600,
                delay: 800,
                easing: 'cubic-bezier(.47,.41,.26,1)',
                fill: 'forwards' 
            });
            signaturePath3.animate(animation, {
                duration: 800,
                delay: 1400,
                easing: 'cubic-bezier(.47,.41,.26,1)',
                fill: 'forwards' 
            });
            signaturePath4.animate(animation, {
                duration: 600,
                delay: 2200,
                easing: 'cubic-bezier(.47,.41,.26,1)',
                fill: 'forwards' 
            });
        });
    }

    onMount(async () => {
        await loadPagePromise;
        introAnimations();
    });

</script>



<div class="footer-wrapper" bind:this={footerContainerElement}>
    <!-- Left side -->
    <div class="flex-wrapper">
        <div class="logo-wrapper">
            <div class="inline-flex" bind:this={logoElement}>
                <img src="assets/imgs/logo.svg" alt="Apurv Ahire logo" class="logo">
            </div>
        </div>

        <div class="status-wrapper">
            {#if dataState.siteData}
                {#if dataState.siteData!.availablity_date === ""}
                    <p class="large-text" bind:this={statusElement}>
                        i am currently accepting freelance work, <br>you may reach me on my email.
                    </p>
                {:else}
                    <p class="large-text" bind:this={statusElement}>
                        i am available for freelance work after <br> {dataState.siteData.availablity_date}.
                    </p>
                {/if}
            {/if}
            <a class="button large-text" bind:this={fullEmailLinkElement} href="mailto:apuravahire2003@gmail.com" target="_blank">apuravahire2003@gmail.com</a>
        </div>
        
        <div class="credits-wrapper" bind:this={creditsElement}>
            <p class="year">© {currentYear}</p>
            <p class="credits">
                designed and developed by Apurv Ahire<br>
                
                <!-- Support the project by keeping this line in your fork -->
                <a class="clickable button no-decor" href="https://github.com/ApurvAhire03" target="_blank">
                    this website is open source on github
                </a>
            </p>
        </div>
    </div>

    <!-- Right side -->
	<div class="flex-wrapper decor">
        <!-- Apurv Ahire SVG Signature -->
        <svg id="signature" class="name-signature" x="0px" y="0px" viewBox="0 0 240 120" style="stroke: rgb(79, 78, 85);">
            <g>
                <path
                    bind:this={signaturePath1}
                    class="path-1"
                    style="fill:none;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
                    d="M 22 75 C 24 64 28 42 38 22 C 43 12 49 13 52 20 C 55 35 55 58 53 78 C 51 86 44 85 40 76 C 35 64 42 51 54 52 C 63 53 71 58 76 63"/>
                <path
                    bind:this={signaturePath2}
                    class="path-2"
                    style="fill:none;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
                    d="M 76 63 C 80 57 84 51 88 50 C 89 53 88 78 87 100 C 86 106 79 104 80 92 C 81 78 84 54 89 52 C 95 50 102 54 101 64 C 100 72 92 76 87 75 C 91 75 96 73 103 67"/>
                <path
                    bind:this={signaturePath3}
                    class="path-3"
                    style="fill:none;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
                    d="M 103 67 C 107 61 111 53 116 52 C 118 56 119 69 124 73 C 128 75 132 60 135 53 C 137 57 138 69 143 73 C 146 75 149 55 154 52 C 158 50 161 53 162 57 C 163 64 165 71 169 74 C 172 75 175 57 179 54 C 181 58 184 71 188 73 C 193 74 198 63 203 53 C 205 49 209 49 212 53"/>
                <path
                    bind:this={signaturePath4}
                    class="path-4"
                    style="fill:none;stroke-width:2.5;stroke-linecap:round;stroke-linejoin:round;stroke-opacity:1;stroke-miterlimit:4;"
                    d="M 28 92 C 55 95 105 94 155 89 C 182 86 205 81 220 74"/>
            </g>
        </svg>
    </div>
</div>



<style lang="sass">

@use "../consts.sass" as consts

@include consts.textStyles()

.footer-wrapper
    width: 100vw
    background-color: #131314
    display: flex
    flex-direction: row
    justify-content: space-between
    padding: 15vh 13vw
    margin-top: 25vh
    box-sizing: border-box

    @media only screen and (max-width: 950px)
        flex-direction: column-reverse
        padding: 8vh 6vw
        margin-top: 14vh

        .flex-wrapper:not(:first-child)
            margin-bottom: 6vh

        .flex-wrapper.decor
            display: none !important

    .inline-flex
        flex-grow: 1
        display: flex
        flex-direction: row
        align-items: center


    .logo-wrapper
        margin-bottom: 4vh

        .logo
            display: inline-block
            height: clamp(36px, 5vh, 50px)

    .status-wrapper
        .button.large-text
            margin-top: 2vh
            word-break: break-all

    .credits-wrapper
        margin-top: 4vh
        color: rgba(255,255,255,0.4)

        p.year
            margin-bottom: 1vh
            font-family: consts.$font
            font-size: clamp(0.85rem, 1.6vh, 1rem)
            font-weight: normal
            color: rgba(255,255,255,0.4)

        .credits
            font-size: clamp(0.8rem, 1.4vh, 0.95rem)
            line-height: 140%
            white-space: normal
            color: rgba(255,255,255,0.4)

            .button
                color: rgba(255,255,255,0.4)

    .large-text
        font-size: clamp(1.15rem, 2.4vh, 1.6rem)
        line-height: 140%

        @media only screen and (max-width: 950px)
            br
                display: none

    .flex-wrapper.decor
        display: flex
        flex-direction: column
        justify-content: center

        .name-signature
            width: 20vh

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

</style>