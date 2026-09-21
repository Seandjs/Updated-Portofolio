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
	<header class="fixed inset-x-0 z-10 mx-auto my-6">
		<nav class="flex flex-row items-center justify-center gap-4">
			<div class="items-center justify-center rounded-full shadow-sm">
				<a href="/" aria-label="Westala">
					<div
						class="relative m-2 h-10 w-10 bg-(--dark-color) mask-[url('/logo.svg')] mask-contain mask-no-repeat transition-all ease-in-out hover:animate-spin"
					></div>
				</a>
			</div>
			<div class="font-ubuntu font-base flex gap-10 rounded-xl p-4 px-20 text-sm shadow-sm">
				<a href="/" class="nav-link"> About </a>
				<a href="/" class="nav-link"> Work </a>
				<a href="/" class="nav-link"> Experience </a>
			</div>
			<div class="font-ubuntu flex gap-5 rounded-4xl px-4 py-5 text-base font-light shadow-sm">
				<i class="fa-solid fa-moon"></i>
				<i class="fa-solid fa-sun"></i>
			</div>
		</nav>
	</header>
	<main class="grow">
		{@render children()}
	</main>
	<footer class="bg-(--dark-color) text-(--light-color)">
		<div>
			<div>
				<a href="/">Github</a>
				<a href="/">Instagram</a>
			</div>
			<div></div>
		</div>
	</footer>
</div>
