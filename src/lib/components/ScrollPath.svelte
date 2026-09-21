<!--
  ScrollPath.svelte  →  src/lib/components/ScrollPath.svelte

  Membungkus SATU section (hero). Saat user scroll:
    1. Section ini di-"pin" (tetap diam di layar).
    2. Garis tergambar mengikuti scroll (scrub).
    3. Setelah garis selesai, pin dilepas dan halaman lanjut ke section berikutnya.

  Pemakaian (+page.svelte):
    <ScrollPath>
      <section id="hero">…</section>
    </ScrollPath>
    <section id="highlight">…</section>
-->
<script>
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	const DEFAULT_D =
		'M -240 220 C -60 120, 140 62, 290 68 C 370 74, 400 100, 340 150 ' +
		'C 280 200, 150 240, 150 300 C 150 340, 200 360, 260 355 ' +
		'C 340 350, 420 300, 500 285 C 560 275, 560 320, 510 385 ' +
		'C 450 470, 420 520, 420 590 C 420 680, 560 740, 760 760 ' +
		'C 1000 785, 1300 730, 1500 680 C 1650 640, 1800 590, 1950 530';

	let {
		d = DEFAULT_D,
		viewBox = '0 0 1900 900',
		color = 'var(--primary-color)',
		strokeWidth = 90,
		initial = 0.04,
		distance = '+=150%',
		scrub = 1,
		respectReducedMotion = false,
		children
	} = $props();

	let root = $state();
	let pathEl = $state();

	onMount(() => {
		if (respectReducedMotion && window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
			pathEl.setAttribute('stroke-dashoffset', '0');
			return;
		}

		gsap.registerPlugin(ScrollTrigger);

		ScrollTrigger.clearScrollMemory('manual');

		const ctx = gsap.context(() => {
			gsap.fromTo(
				pathEl,
				{ attr: { 'stroke-dashoffset': 1 - initial } },
				{
					attr: { 'stroke-dashoffset': 0 },
					ease: 'none', // WAJIB linear supaya garis pas mengikuti scrollbar
					scrollTrigger: {
						trigger: root,
						start: 'top top', // mulai saat hero menempel di atas layar
						end: distance, // selesai setelah user scroll sejauh `distance`
						pin: root, // ← INI yang menahan hero di layar selama menggambar
						pinSpacing: true, // beri ruang scroll ekstra agar section berikutnya tidak tertimpa
						scrub,
						invalidateOnRefresh: true
					}
				}
			);
		}, root);

		// Font web mengubah tinggi layout; hitung ulang posisi pin setelah font siap.
		document.fonts?.ready.then(() => ScrollTrigger.refresh());

		return () => ctx.revert();
	});
</script>

<!-- min-h-screen: tinggi = 1 layar penuh, cocok dengan hero-mu -->
<div bind:this={root} class="relative min-h-screen">
	<svg
		class="pointer-events-none absolute inset-0 z-0 h-full w-full"
		{viewBox}
		preserveAspectRatio="xMidYMid slice"
		aria-hidden="true"
	>
		<path
			bind:this={pathEl}
			{d}
			pathLength="1"
			fill="none"
			style:stroke={color}
			stroke-width={strokeWidth}
			stroke-linecap="round"
			stroke-linejoin="round"
			stroke-dasharray="1"
			stroke-dashoffset="1"
		/>
	</svg>

	<!-- Konten di atas garis (z-1), tapi di bawah header fixed (z-10) -->
	<div class="relative z-1 flex min-h-screen flex-col">
		{@render children?.()}
	</div>
</div>
