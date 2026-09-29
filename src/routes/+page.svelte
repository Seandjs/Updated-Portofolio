<script>
	import ScrollPath from '$lib/components/ScrollPath.svelte';
	import VisionSection from '$lib/components/VisionSection.svelte';
	import './layout.css';
	import '../app.css';
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { EasePack } from 'gsap/EasePack';
	gsap.registerPlugin(EasePack);

	let headerIntro;

	function playHeroAnimation() {
		gsap.to(headerIntro.children, {
			opacity: 1,
			y: 0,
			duration: 0.8,
			stagger: 0.15,
			ease: 'power3.out'
		});
	}
	onMount(() => {
		window.addEventListener('preloaderFinished', playHeroAnimation);
		if ('scrollRestoration' in history) history.scrollRestoration = 'manual';
		window.scrollTo(0, 0);
		const toTop = () => window.scrollTo(0, 0);
		window.addEventListener('beforeunload', toTop);
	});

	const modules = import.meta.glob('$lib/assets/visionsection/*.{jpg,jpeg,png,webp}', {
		eager: true,
		import: 'default'
	});
	const photos = Object.values(modules);
</script>

<svelte:head>
	<title>Dhefano Seandy</title>
</svelte:head>

<ScrollPath color="var(--primary-color)" opacity={0.6}>
	<section
		id="hero"
		class="inset-0 flex max-h-screen min-h-screen flex-col items-center justify-center gap-6"
		bind:this={headerIntro}
	>
		<p class="translate-y-8 text-xl font-light opacity-0">¡Hola Amigos! soy</p>
		<h1 class="translate-y-8 text-6xl font-bold opacity-0">
			<span class="font-sacramento font-black">D</span>hefano
		</h1>
		<p class="-my-5 -mt-8 translate-y-8 font-light opacity-0">as a</p>
		<h2 class="translate-y-8 text-3xl opacity-0">Software Engineering Student</h2>
		<button class="mt-5 translate-y-8 opacity-0">
			<a
				href="/"
				class="group rounded-3xl border border-(--primary-color) px-3 py-2 text-sm text-(--primary-color)"
				>About me <i
					class="fa-regular fa-hand-point-up ms-1 inline-block rotate-90 transition-all ease-in-out"
				></i></a
			>
		</button>
	</section>
</ScrollPath>
<VisionSection images={photos} columns={4} />
<section id="highlight" class="flex min-h-screen flex-col items-center pt-25">
	<div class="flex w-full max-w-[90%] flex-col gap-12">
		<div class="font-montserrat-alt flex flex-col gap-1">
			<span class="w-fit rounded-full border px-2 py-1.5 text-base"> Highlight </span>
			<h2 class="text-5xl font-bold">Experience</h2>
		</div>
		<div class="text-base">
			<a
				href="/"
				class="group grid grid-cols-3 items-center border-[1.5] border-y py-7 opacity-70 transition-all duration-400 ease-in-out hover:opacity-100"
			>
				<span class="text-left"
					>Acara A <i
						class="fa-regular fa-hand-point-up ms-2 inline-block rotate-90 opacity-0 duration-400 group-hover:opacity-100"
					></i></span
				><span class="text-center">Kategori</span><span class="text-right">2026</span>
			</a>
		</div>
		<div class="flex items-center justify-center">
			<a
				href="/"
				class="group rounded-3xl border border-(--primary-color) px-3 py-2 text-sm text-(--primary-color) transition-all ease-in-out hover:scale-105 hover:border-(--secondary-color) hover:text-(--secondary-color) hover:shadow-xs duration-180"
				>More <i
					class="fa-regular fa-hand-point-up ms-1 inline-block rotate-90 transition-all ease-in-out"
				></i></a
			>
		</div>
	</div>
</section>
