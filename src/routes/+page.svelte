<script>
    import Quote from '#lib/components/Quote.svelte';
</script>

<svelte:head>
    <title>Words Worth Keeping | Data Exchange</title>
    <meta name="description" content="Five inspiring quotes to copy and share." />
</svelte:head>

<main class="min-h-screen bg-[#11121b] px-5 py-12 text-white sm:px-8 lg:py-20">

    <div class="mx-auto max-w-6xl">

        <!-- Header -->
        <header class="mb-10 flex flex-col gap-6 border-b border-white/10 pb-10 sm:flex-row sm:items-end sm:justify-between">

            <div>
                <p class="text-xs font-bold uppercase tracking-[0.3em] text-violet-300">
                    The Quote Archive ✦
                </p>

                <h1 class="mt-5 font-serif text-5xl leading-tight tracking-tight sm:text-6xl">
                    Words worth <span class="italic text-violet-300">keeping.</span>
                </h1>

                <p class="mt-4 max-w-lg text-sm leading-6 text-slate-400 sm:text-base">
                    Five little reminders for the days you need inspiration.
                    Find one you love and pass it on.
                </p>
            </div>

            <div class="w-fit rounded-2xl border border-violet-300/20 bg-violet-300/10 px-5 py-4">
                <span class="text-3xl font-bold text-violet-200">05</span>
                <span class="ml-2 text-sm text-slate-300">quotes to explore</span>
            </div>

        </header>


        <!-- Quotes -->
        <section aria-label="Quotes" class="grid gap-5 md:grid-cols-2 xl:grid-cols-3">

            <Quote number="01" author="Steve Jobs">
                The only way to do great work is to love what you do.
            </Quote>

            <Quote number="02" author="Maya Angelou">
                Nothing will work unless you do.
            </Quote>

            <Quote number="03" author="Benjamin Franklin">
                Well done is better than well said.
            </Quote>

            <Quote number="04" author="Theodore Roosevelt">
                Do what you can, with what you have, where you are.
            </Quote>

            <Quote number="05" author="Helen Keller">
                Alone we can do so little; together we can do so much.
            </Quote>

        </section>


        <!-- Footer -->
        <footer class="mt-12 text-center text-xs tracking-wide text-slate-500">
            A little inspiration, one quote at a time.
        </footer>

    </div>

</main>