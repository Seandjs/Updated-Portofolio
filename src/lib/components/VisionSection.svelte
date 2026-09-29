<!--
  VisionSectionElegant.svelte  →  src/lib/components/VisionSectionElegant.svelte

  Konsep: "Dim Overlay" + "Column Carousel" — overlay gelap menutupi grid
  gambar di belakangnya selagi section di-pin, tagline muncul di tengah baris
  demi baris, kata menyala satu-satu, lalu logo kecil aksen. Setelah timeline
  selesai, section unpin dan scroll lanjut normal.

  CATATAN:
    - Gambar background diambil OTOMATIS dari folder
      src/lib/assets/visionsection/ (jpg, jpeg, png, webp). Tinggal taruh
      file di folder itu, tidak perlu edit kode.
    - Tiap kolom LOOP MULUS: isi kolom diduplikasi 2x jadi satu strip panjang,
      lalu posisinya di-wrap pakai modulo (gsap.utils.wrap) dalam rentang [-50, 0].
    - Gerak kolom ada DI DALAM timeline (lewat objek proxy), jadi ikut
      dihaluskan oleh `scrub` dan tidak patah-patah mengikuti wheel mouse.
    - Kolom index genap → gambar bergerak ke BAWAH, index ganjil → ke ATAS.

  Pemakaian:
    <VisionSectionElegant />

    Atau override manual:
    <VisionSectionElegant
      images={['/foto1.jpg', '/foto2.jpg']}
      columns={4}
      lines={[['designing', 'with', 'intent,'], ['creating', 'with', 'passion.']]}
      logo="/logo.svg"
    />
-->
<script>
	import { onMount, untrack } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	// Ambil semua gambar dari folder secara otomatis (jpg, jpeg, png, webp).
	// Catatan: pola glob harus berupa string literal, tidak bisa dari variabel/prop.
	const imageModules = import.meta.glob(
		'/src/lib/assets/visionsection/*.{jpg,jpeg,png,webp,JPG,JPEG,PNG,WEBP}',
		{ eager: true, query: '?url', import: 'default' }
	);
	const folderImages = Object.values(imageModules);

	let {
		lines = [
			['is', 'this', 'the'],
			['world', 'we', 'created?']
		],
		logo = '/logo.svg', // logo kecil di akhir teks, null untuk mematikan
		images = folderImages, // default: otomatis dari folder; tetap bisa di-override lewat prop
		columns = 4, // jumlah kolom grid background
		tilesPerColumn = 6, // minimal gambar per kolom sebelum di-cycle
		shuffle = true, // acak urutan & sebaran foto setiap kali komponen dimuat
		travelPercent = 40, // jarak "jalan" strip sepanjang pin (% tinggi 1 set gambar)
		scrubSmooth = 1.2, // makin besar makin halus/lembam (turunkan ke ~0.5 kalau pakai Lenis)
		gridOpacity = 0.75,
		overlayColor = 'var(--dark-color)',
		overlayOpacity = 0.75,
		textColor = 'var(--light-color)',
		distance = '+=300%',
		respectReducedMotion = false
	} = $props();

	let wrap = $state();
	let overlay = $state();

	// Fisher-Yates shuffle — tanpa mengubah array aslinya.
	function shuffleArray(arr) {
		const a = [...arr];
		for (let i = a.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			[a[i], a[j]] = [a[j], a[i]];
		}
		return a;
	}

	// Tiap kolom mengambil sample acaknya sendiri dari seluruh pool `images`.
	function buildColumn() {
		// Pengaman: folder kosong → hindari loop tanpa akhir.
		if (!images.length) return [];

		let col = [];
		while (col.length < tilesPerColumn) {
			col = col.concat(shuffle ? shuffleArray(images) : images);
		}
		col = col.slice(0, tilesPerColumn);
		return [...col, ...col]; // duplikat → strip loop mulus
	}

	// Pengacakan cukup sekali saat dimuat; sengaja hanya baca nilai awal props.
	const columnList = untrack(() => Array.from({ length: columns }, () => buildColumn()));

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);

		const ctx = gsap.context(() => {
			const colInners = gsap.utils.toArray('.parallax-col-inner');

			// Strip diulang tiap 50% (2 set gambar identik). yPercent harus selalu
			// di rentang [-50, 0] supaya bagian atas kolom tidak pernah kosong.
			const wrapRange = gsap.utils.wrap(-50, 0);

			// Posisi semua kolom untuk progress p (0 → 1).
			const applyColumns = (p) => {
				const raw = p * travelPercent;
				colInners.forEach((el, i) => {
					const isDown = i % 2 === 0;
					gsap.set(el, {
						yPercent: isDown ? wrapRange(raw - travelPercent) : wrapRange(-raw)
					});
				});
			};

			// Posisi awal, biar tidak ada lompatan di frame pertama.
			applyColumns(0);

			if (respectReducedMotion && window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
				gsap.set(overlay, { opacity: overlayOpacity });
				gsap.set('.word, .line, .logo-mark', { opacity: 1, y: 0, yPercent: 0, scale: 1 });
				return;
			}

			const tl = gsap.timeline({
				defaults: { ease: 'none' },
				scrollTrigger: {
					trigger: wrap,
					start: 'top top',
					end: distance,
					pin: true,
					pinSpacing: true,
					scrub: scrubSmooth,
					anticipatePin: 1,
					invalidateOnRefresh: true
				}
			});

			// 1) Overlay gelap menutupi background (0 → 1)
			tl.fromTo(overlay, { opacity: 0 }, { opacity: overlayOpacity, duration: 1 }, 0);

			// 2) Baris naik pelan dari bawah, satu demi satu (0.6 → 1.8)
			gsap.utils.toArray('.line').forEach((el, i) => {
				tl.fromTo(
					el,
					{ yPercent: 45, opacity: 0 },
					{ yPercent: 0, opacity: 1, ease: 'power2.out', duration: 1 },
					0.6 + i * 0.2
				);
			});

			// 3) Kata menyala satu per satu (1.4 → ±3)
			tl.fromTo('.word', { opacity: 0.18 }, { opacity: 1, duration: 0.35, stagger: 0.22 }, 1.4);

			// 4) Logo kecil scale-in sebagai aksen (2.8 → 3.2)
			if (logo) {
				tl.fromTo(
					'.logo-mark',
					{ scale: 0, opacity: 0 },
					{ scale: 1, opacity: 1, ease: 'back.out(2.2)', duration: 0.4 },
					2.8
				);
			}

			// 5) Tahan sejenak sebelum unpin
			tl.to({}, { duration: 0.6 });

			// 6) Carousel kolom: proxy tween sepanjang timeline, jadi ikut
			//    dihaluskan oleh scrub (bukan dibaca mentah dari posisi scroll).
			const proxy = { p: 0 };
			tl.to(
				proxy,
				{
					p: 1,
					duration: tl.duration(),
					ease: 'none',
					onUpdate: () => applyColumns(proxy.p)
				},
				0
			);
		}, wrap);

		return () => ctx.revert();
	});
</script>

<div bind:this={wrap} class="relative h-screen w-full overflow-hidden bg-(--dark-color)">
	<!-- Background: grid berkolom, tiap kolom strip gambar loop mulus -->
	<div
		class="absolute inset-0 grid gap-4 p-8"
		style:grid-template-columns={`repeat(${columns}, minmax(0, 1fr))`}
		style:opacity={gridOpacity}
		aria-hidden="true"
	>
		{#each columnList as col, ci}
			<div class="parallax-col relative h-full w-full overflow-hidden">
				<div
					class="parallax-col-inner col-{ci} absolute inset-x-0 top-0 flex flex-col gap-4 will-change-transform"
				>
					{#each col as src}
						<img
							{src}
							alt=""
							class="h-auto w-full shrink-0 rounded-xl object-cover"
							loading="eager"
							decoding="async"
						/>
					{/each}
				</div>
			</div>
		{/each}
	</div>

	<!-- Overlay gelap -->
	<div
		bind:this={overlay}
		class="pointer-events-none absolute inset-0 opacity-0"
		style:background={overlayColor}
	></div>

	<!-- Teks tagline -->
	<div
		class="relative z-1 flex h-full items-center justify-center px-6 text-center"
		style:color={textColor}
	>
		<h2
			class="text-[clamp(1.1rem,5.5vw,4.5rem)] leading-tight font-medium tracking-tight text-(--light-color)"
		>
			{#each lines as words, i}
				<span class="line line-{i} block overflow-hidden">
					<span class="block py-1">
						{#each words as w, wi}
							<span class="word me-[0.25em] inline-block">
								{w}
								{#if logo && i === lines.length - 1 && wi === words.length - 1}
									<span
										class="logo-mark ms-1 inline-block h-[0.7em] w-[0.7em] rounded-full border border-(--primary-color) bg-(--light-color) p-2 align-middle"
									>
										<div
											class="h-full w-full bg-(--primary-color) mask-contain mask-no-repeat"
											style:mask-image={`url('${logo}')`}
											style:-webkit-mask-image={`url('${logo}')`}
										></div>
									</span>
								{/if}
							</span>
						{/each}
					</span>
				</span>
			{/each}
		</h2>
	</div>
</div>