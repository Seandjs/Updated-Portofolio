<script>
	import './layout.css';
	import '../app.css';
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { EasePack } from 'gsap/EasePack';
	gsap.registerPlugin(EasePack);

	let { children } = $props();
	let loaderOverlayFront;
	let loaderOverlayBack;
	let ballRef;
	let mainContent;

	onMount(() => {
		const tl = gsap.timeline({ defaults: { ease: 'power3.inOut' } });

		tl.from(ballRef, {
			x: -1000,
			duration: 2,
			// ease: 'bounce.out',
			repeat: -1,
			repeatDelay: 0.5
		})
			.to(loaderOverlayFront, {
				yPercent: -100,
				duration: 1,
				ease: 'power4.inOut'
			})
			.to(
				loaderOverlayBack,
				{
					yPercent: -100,
					duration: 1.4,
					ease: 'power4.inOut'
				},
				'<'
			);
	});
</script>

<svelte:head>
	<title>Dhefano Seandy</title>
</svelte:head>

<div
	bind:this={loaderOverlayBack}
	class="fixed inset-0 z-100 flex items-center justify-center bg-[#f1ac7b]"
>
	<div
		bind:this={loaderOverlayFront}
		class="fixed inset-0 flex items-center justify-center bg-(--primary-color)"
	>
		<div bind:this={ballRef} class="rounded-full border-2 border-(--dark-color) bg-(--light-color)">
			<div
				class="relative m-4 h-25 w-25 animate-[spin_2.3s_linear_infinite] bg-(--dark-color) mask-[url('/logo.svg')] mask-contain mask-no-repeat"
			></div>
		</div>
	</div>
</div>

<div class="flex min-h-screen flex-col bg-(--light-color) text-(--dark-color)">
	<header class="fixed inset-x-0 mx-auto my-6">
		<nav class="flex flex-row items-center justify-center gap-4">
			<div class="items-center justify-center rounded-full shadow-sm">
				<a href="/" aria-label="Westala">
					<div
						class="relative m-2 h-10 w-10 bg-(--dark-color) mask-[url('/logo.svg')] mask-contain mask-no-repeat transition-all ease-in-out hover:animate-spin"
					></div>
				</a>
			</div>
			<div class="font-ubuntu flex gap-10 rounded-xl p-4 px-20 text-sm font-base shadow-sm">
				<a href="/" class="transition-all ease-in-out hover:scale-110"> Home </a>
				<a href="/" class="transition-all ease-in-out hover:scale-110"> About </a>
				<a href="/" class="transition-all ease-in-out hover:scale-110"> Work </a>
				<a href="/" class="transition-all ease-in-out hover:scale-110"> Experience </a>
			</div>
			<div class="font-ubuntu flex gap-5 rounded-4xl py-5 px-4  text-base font-light shadow-sm">
				<i class="fa-solid fa-moon"></i>
				<i class="fa-solid fa-sun"></i>
			</div>
		</nav>
	</header>
	<main class="grow">
		<!-- {@render children()} -->
	</main>
	<footer >

	</footer>
</div>
