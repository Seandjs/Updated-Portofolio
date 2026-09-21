<!--
  VisionSectionElegant.svelte  →  src/lib/components/VisionSectionElegant.svelte

  Varian "art deco tenang": bingkai garis tipis, sunburst separuh lingkaran,
  cincin ganda, dan permata (lozenge) kecil. Semua garis tipis, warna solid,
  hanya memakai --primary-color, --light-color, --dark-color.
  Tanpa gradient, tanpa blur.

  Alur scroll sama seperti versi sebelumnya:
    1. Panel membesar
    2. Latar terang → gelap, bingkai & ornamen muncul lembut
    3. Baris 1 dari kiri, baris 2 dari kanan
    4. Kata menyala satu per satu
    5. Bola jatuh memantul jadi titik di akhir kalimat

  Pemakaian:
    <VisionSectionElegant />
    <VisionSectionElegant frame={false} />
-->
<script>
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	// type    : 'sunburst' | 'rings' | 'lozenge' | 'rule'
	// size    : LEBAR bentuk, % dari lebar panel
	// top/left: posisi pojok kiri-atas, % dari panel
	// color   : var(--primary-color) | var(--light-color) | var(--dark-color)
	// opacity : 0–1 (default 1)
	// mobile  : false = disembunyikan di layar kecil
	const defaultShapes = [
		// Sunburst: sinar garis tipis, separuh terpotong di tepi bawah panel
		{ type: 'sunburst', size: '46%', top: '80%', left: '27%', color: 'var(--primary-color)' },
		// Dua cincin tipis terpotong di pojok kanan-atas
		{
			type: 'rings',
			size: '24%',
			top: '-12%',
			left: '84%',
			color: 'var(--light-color)',
			opacity: 0.35
		},
		// Garis dengan permata di tengah, di atas teks
		{
			type: 'rule',
			size: '12%',
			top: '11%',
			left: '44%',
			color: 'var(--light-color)',
			opacity: 0.55,
			mobile: false
		},
		// Dua permata kecil sebagai penyeimbang
		{ type: 'lozenge', size: '2.2%', top: '32%', left: '6%', color: 'var(--primary-color)' },
		{
			type: 'lozenge',
			size: '2.2%',
			top: '66%',
			left: '91%',
			color: 'var(--primary-color)',
			mobile: false
		}
	];

	let {
		lines = [['to', 'infinity'], ['and beyond']],
		shapes = defaultShapes,
		frame = true, // bingkai garis tipis di dalam panel
		darkBg = 'var(--dark-color)', // warna solid
		distance = '+=300%',
		respectReducedMotion = false
	} = $props();

	let wrap = $state();
	let card = $state();
	let darkLayer = $state();
	let textWrap = $state();

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);

		const css = getComputedStyle(document.documentElement);
		const readVar = (name, fallback) => css.getPropertyValue(name).trim() || fallback;
		const lightColor = readVar('--light-color', '#f5ebdc');

		const ctx = gsap.context(() => {
			if (respectReducedMotion && window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
				gsap.set(darkLayer, { opacity: 1 });
				gsap.set(textWrap, { color: lightColor });
				gsap.set('.word, .ball, .shape, .frame, .ray', { opacity: 1, scale: 1 });
				return;
			}

			// Gerak sangat halus: ornamen bergeser beberapa piksel saja
			gsap.utils.toArray('.shape').forEach((el) => {
				gsap.to(el, {
					y: gsap.utils.random(-5, 5),
					duration: gsap.utils.random(6, 8),
					ease: 'sine.inOut',
					repeat: -1,
					yoyo: true
				});
			});

			const tl = gsap.timeline({
				defaults: { ease: 'none' },
				scrollTrigger: {
					trigger: wrap,
					start: 'top top',
					end: distance,
					pin: true,
					pinSpacing: true,
					scrub: 1,
					anticipatePin: 1,
					invalidateOnRefresh: true
				}
			});

			// 1) Kartu membesar (0 → 1)
			tl.fromTo(card, { scale: 0.88, borderRadius: 32 }, { scale: 1, duration: 1 }, 0);

			// 2) Latar menggelap & teks jadi terang (0,4 → 1,6)
			tl.to(darkLayer, { opacity: 1, duration: 1.2 }, 0.4);
			tl.to(textWrap, { color: lightColor, duration: 1.2 }, 0.4);

			// Bingkai muncul pelan
			tl.fromTo(
				'.frame',
				{ opacity: 0, scale: 0.97 },
				{ opacity: 1, scale: 1, ease: 'power2.out', duration: 1 },
				0.8
			);

			// Ornamen fade-in lembut, bergantian
			tl.fromTo(
				'.shape',
				{ opacity: 0, scale: 0.9 },
				{ opacity: 1, scale: 1, ease: 'power2.out', duration: 0.8, stagger: 0.18 },
				0.9
			);

			// Sinar sunburst menyala satu per satu
			tl.fromTo('.ray', { opacity: 0 }, { opacity: 1, duration: 0.25, stagger: 0.06 }, 1.1);

			// 3) Baris masuk dari sisi berlawanan (0,8 → 2,0)
			tl.fromTo(
				'.line-0',
				{ xPercent: -40 },
				{ xPercent: 0, ease: 'power2.out', duration: 1.2 },
				0.8
			);
			tl.fromTo(
				'.line-1',
				{ xPercent: 40 },
				{ xPercent: 0, ease: 'power2.out', duration: 1.2 },
				0.8
			);

			// 4) Kata menyala satu per satu (1,8 → ±3,5)
			tl.fromTo('.word', { opacity: 0.15 }, { opacity: 1, duration: 0.4, stagger: 0.25 }, 1.8);

			// 5) Bola jatuh & memantul jadi titik (3,5 → 4,5)
			tl.fromTo(
				'.ball',
				{ y: () => -card.offsetHeight * 0.7, opacity: 0 },
				{ y: 0, opacity: 1, ease: 'bounce.out', duration: 1 },
				3.5
			);

			tl.to({}, { duration: 0.5 });
		}, wrap);

		document.fonts?.ready.then(() => ScrollTrigger.refresh());

		return () => ctx.revert();
	});
</script>

<div bind:this={wrap} class="relative flex h-screen w-full items-center justify-center border-y-8">
	<section
		id="middle"
		bind:this={card}
		aria-label="Visi"
		class="relative w-[95%] h-[90%] overflow-hidden rounded-4xl bg-(--light-color) shadow-md"
	>
		<!-- Lapisan gelap solid: opacity 0 → 1 lewat GSAP -->
		<div bind:this={darkLayer} class="absolute inset-0 opacity-0" style:background={darkBg}></div>

		<!-- Bingkai garis tipis ganda (border + outline), warna terang -->
		{#if frame}
			<div
				class="frame pointer-events-none absolute inset-[3.5%] rounded-3xl border opacity-0"
				style="border-color: var(--light-color); outline: 1px solid var(--light-color); outline-offset: 6px; --tw-opacity: 1;"
				aria-hidden="true"
			></div>
		{/if}

		<!-- Ornamen: hiasan saja, flat, warna via currentColor -->
		<div class="pointer-events-none absolute inset-0" aria-hidden="true">
			{#each shapes as s}
				<div
					class="shape absolute {s.mobile === false ? 'max-md:hidden' : ''}"
					style:width={s.size}
					style:top={s.top}
					style:left={s.left}
					style:color={s.color}
				>
					<div style:opacity={s.opacity ?? 1}>
						{#if s.type === 'sunburst'}
							<!-- 13 sinar dari titik pusat bawah, sudut 0°–180° -->
							<svg viewBox="0 0 200 100" class="block h-auto w-full overflow-visible">
								{#each Array(13) as _, k}
									{@const a = (k * 15 * Math.PI) / 180}
									<line
										class="ray"
										x1={100 + 28 * Math.cos(a)}
										y1={100 - 28 * Math.sin(a)}
										x2={100 + 96 * Math.cos(a)}
										y2={100 - 96 * Math.sin(a)}
										stroke="currentColor"
										stroke-width="1.6"
										stroke-linecap="round"
									/>
								{/each}
								<path
									d="M72 100 A28 28 0 0 1 128 100"
									fill="none"
									stroke="currentColor"
									stroke-width="1.6"
								/>
							</svg>
						{:else if s.type === 'rings'}
							<svg viewBox="0 0 100 100" class="block h-auto w-full overflow-visible">
								<circle
									cx="50"
									cy="50"
									r="48"
									fill="none"
									stroke="currentColor"
									stroke-width="1.2"
								/>
								<circle
									cx="50"
									cy="50"
									r="38"
									fill="none"
									stroke="currentColor"
									stroke-width="1.2"
								/>
							</svg>
						{:else if s.type === 'rule'}
							<svg viewBox="0 0 100 10" class="block h-auto w-full overflow-visible">
								<line x1="0" y1="5" x2="42" y2="5" stroke="currentColor" stroke-width="0.8" />
								<path d="M50 1 L54 5 L50 9 L46 5 Z" fill="currentColor" />
								<line x1="58" y1="5" x2="100" y2="5" stroke="currentColor" stroke-width="0.8" />
							</svg>
						{:else if s.type === 'lozenge'}
							<svg viewBox="0 0 100 100" class="block h-auto w-full overflow-visible">
								<path d="M50 0 L100 50 L50 100 L0 50 Z" fill="currentColor" />
							</svg>
						{/if}
					</div>
				</div>
			{/each}
		</div>

		<!-- Teks visi -->
		<div
			bind:this={textWrap}
			class="relative z-1 flex h-full items-center justify-center px-6 text-(--dark-color)"
		>
			<h2 class="text-[clamp(0.75rem,7vw,6.5rem)] leading-none font-medium tracking-tight">
				{#each lines as words, i}
					<span class="line line-{i} block {i % 2 ? 'ps-[1.2em]' : ''}">
						{#each words as w}
							<span class="word me-[0.25em] inline-block">{w}</span>
						{/each}
						{#if i === lines.length - 1}
							<span
								class="ball ms-[0.02em] inline-block aspect-square h-[0.17em] rounded-full bg-(--primary-color)"
							></span>
						{/if}
					</span>
				{/each}
			</h2>
		</div>
	</section>
</div>
