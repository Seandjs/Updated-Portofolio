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
	let navLink = $state([]);
	let mainContent;

	onMount(() => {
		const tl = gsap.timeline({
			defaults: { ease: 'power3.inOut' },
			onComplete: () => window.dispatchEvent(new CustomEvent('preloaderFinished'))
		});

		tl.from(ballRef, {
			x: -1000,
			duration: 2,
			ease: 'expo.inOut',
			// repeat: -1,
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
		const nav = document.querySelectorAll('.nav-link');
		nav.forEach((link) => {
			link.addEventListener('mouseenter', () => {
				gsap.to(link, {
					y: -5,
					duration: 0.3,
					ease: 'power2.out'
				});
			});
			link.addEventListener('mouseleave', () => {
				gsap.to(link, {
					y: 0,
					duration: 0.3,
					ease: 'power2.out'
				});
			});
		});
	});
</script>

<svelte:head>
	<title>Dhefano Seandy</title>
</svelte:head>

<div
	bind:this={loaderOverlayBack}
	class="fixed inset-0 z-100 flex items-center justify-center bg-(--secondary-color)"
>
	<div
		bind:this={loaderOverlayFront}
		class="fixed inset-0 flex items-center justify-center bg-(--dark-color)"
	>
		<div
			bind:this={ballRef}
			class="rounded-full border-2 border-(--primary-color) bg-(--light-color)"
		>
			<div
				class="relative m-4 h-25 w-25 animate-[spin_2.3s_linear_infinite] bg-(--primary-color) mask-[url('/logo.svg')] mask-contain mask-no-repeat"
			></div>
		</div>
	</div>
</div>

<div class="flex min-h-screen flex-col bg-(--light-color) text-(--dark-color)">
	<header class="fixed inset-x-0 z-10 mx-auto my-6">
		<nav class="flex flex-row items-center justify-center gap-4">
			<div
				class="items-center justify-center rounded-full bg-(--light-color) shadow-sm ring ring-white backdrop-blur-md ring-inset"
			>
				<a href="/" aria-label="Westala">
					<div
						class="relative m-2 h-10 w-10 bg-(--dark-color) mask-[url('/logo.svg')] mask-contain mask-no-repeat transition-all ease-in-out hover:animate-spin"
					></div>
				</a>
			</div>
			<div
				class="font-ubuntu font-base flex gap-10 rounded-xl bg-(--light-color) p-4 px-20 text-sm shadow-sm ring ring-white ring-inset"
			>
				<a href="/" class="nav-link"> About </a>
				<a href="/" class="nav-link"> Work </a>
				<a href="/" class="nav-link"> Experience </a>
			</div>
			<div
				class="font-ubuntu flex gap-5 rounded-4xl bg-(--light-color) px-4 py-5 text-base font-light shadow-sm ring ring-white ring-inset"
			>
				<i class="fa-solid fa-moon"></i>
				<i class="fa-solid fa-sun"></i>
			</div>
		</nav>
	</header>
	<main class="grow">
		{@render children()}
	</main>
	<footer class="relative inset-0 rounded-t-2xl bg-(--dark-color) text-(--light-color) shadow-2xs">
		<div class="mx-auto my-4 grid max-w-[90%] grid-cols-3">
			<div class="text-left flex flex-row gap-2 relative">
				<a href="/"
					 class="flex flex-row hover:text-(--primary-color) transition-all ease-in-out duration-150">Guthib <svg
						xmlns="http://w3.org"
						width="16"
						height="16"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2.5"
						stroke-linecap="round"
						stroke-linejoin="round"
						style="vertical-align: middle; margin-left: 5px;"
					>
						<line x1="7" y1="17" x2="17" y2="7"></line>
						<polyline points="7 7 17 7 17 17"></polyline>
					</svg>
				</a>
				<a href="/"
					 class="flex flex-row hover:text-(--primary-color) transition-all ease-in-out duration-150">Instagram <svg
						xmlns="http://w3.org"
						width="16"
						height="16"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2.5"
						stroke-linecap="round"
						stroke-linejoin="round"
						style="vertical-align: middle; margin-left: 5px;"
					>
						<line x1="7" y1="17" x2="17" y2="7"></line>
						<polyline points="7 7 17 7 17 17"></polyline>
					</svg>
				</a>
			</div>
			<div class="relative flex justify-center text-center items-center inset-0">
				<a href="/" aria-label="Westala" class="bg-(--light-color) border-(--primary-color) border-2 rounded-full">
					<div
						class="relative m-2 h-10 w-10  bg-(--primary-color) mask-[url('/logo.svg')] mask-contain mask-no-repeat transition-all ease-in-out hover:animate-spin"
					></div>
				</a>
			</div>
			<div class="flex flex-col items-end text-right">
				<p>
					©2026 Built & Designed
					<br />
					by <span class="font-medium">Dhefano Seandy</span>
				</p>
			</div>
		</div>
	</footer>
</div>
