<script>
    import { onDestroy } from 'svelte';
    import { getDistance } from 'geolib';
    import Quote from '#lib/components/Quote.svelte';

    const quotes = [
        {
            id: 1,
            author: 'Paul Opitz',
            text: "Don't be Captain Obvious!"
        },
        {
            id: 2,
            author: 'Gandalf',
            text: 'You shall not pass!'
        },
        {
            id: 3,
            author: 'Bilbo Baggins',
            text: 'Not all those who wander are lost.'
        },
        {
            id: 4,
            author: 'Eminem',
            text: 'Hi, my name is, what? My name is, who? My name is, chka-chka, Slim Shady.'
        },
        {
            id: 5,
            author: 'Shkodra Elektronike',
            text: "Njerëzit e mirë e njerëzit pa emër, kërcejnë valle n'shpirt. Jarnane ti toka ime, ku kam lindë s'do të harroj."
        }
    ];

    // Saved quotes
    let savedIds = $state([]);

    function moveQuote(id, toSaved) {
        if (toSaved && !savedIds.includes(id)) {
            savedIds = [...savedIds, id];
        }

        if (!toSaved) {
            savedIds = savedIds.filter((item) => item !== id);
        }
    }

    // Drag and drop
    function dragStart(event, id) {
        event.dataTransfer.setData('text/plain', String(id));
        event.dataTransfer.effectAllowed = 'move';
    }

    function dropQuote(event, toSaved) {
        event.preventDefault();

        const id = Number(event.dataTransfer.getData('text/plain'));

        if (quotes.some((quote) => quote.id === id)) {
            moveQuote(id, toSaved);
        }
    }

    // Easter egg coordinates
    const secretSpot = {
        latitude: 42.05910523134263,
        longitude: 19.517268301553923
    };

    let watchId = null;
    let tracking = $state(false);
    let secretUnlocked = $state(false);
    let locationMessage = $state('A secret quote is hidden somewhere nearby.');

    function stopTracking() {
        if (watchId !== null) {
            navigator.geolocation.clearWatch(watchId);
        }

        watchId = null;
        tracking = false;
    }

    function checkLocation() {
        if (tracking) {
            stopTracking();
            locationMessage = 'Location checking stopped.';
            return;
        }

        if (!navigator.geolocation) {
            locationMessage = 'This browser does not support location access.';
            return;
        }

        tracking = true;
        locationMessage = 'Getting your location...';

        watchId = navigator.geolocation.watchPosition(
            (position) => {
                const distance = getDistance(position.coords, secretSpot);

                if (distance <= 100) {
                    secretUnlocked = true;
                    locationMessage = 'You found the secret!';
                    stopTracking();
                } else {
                    locationMessage =
                        distance < 1000
                            ? `You are ${distance} m from the secret spot.`
                            : `You are ${(distance / 1000).toFixed(1)} km from the secret spot.`;
                }
            },
            (error) => {
                locationMessage =
                    error.code === 1
                        ? 'Location permission was denied. Allow it in your browser settings.'
                        : 'Could not get your location. Try again outdoors.';

                stopTracking();
            },
            {
                enableHighAccuracy: true,
                maximumAge: 0,
                timeout: 20000
            }
        );
    }

    onDestroy(() => {
        if (watchId !== null) {
            stopTracking();
        }
    });
</script>

<svelte:head>
    <title>The Quote Archive</title>
    <meta
        name="description"
        content="Save, copy, share, and discover secret quotes."
    />
</svelte:head>

<main class="min-h-screen bg-slate-50 px-4 py-10 text-slate-900 sm:px-8">

    <div class="mx-auto max-w-6xl">

        <!-- Header -->
        <header class="mb-9">

            <p class="text-xs font-bold uppercase tracking-[0.25em] text-indigo-600">
                ✦ The Quote Archive
            </p>

            <h1 class="mt-3 text-4xl font-bold tracking-tight sm:text-5xl">
                Little words, big meaning.
            </h1>

            <p class="mt-3 text-slate-500">
                Drag quotes between the columns, or use the Save and Return buttons.
            </p>

        </header>

        <!-- Two columns -->
        <div class="grid gap-5 lg:grid-cols-2">

            <!-- LEFT: Explore -->
            <section
                aria-label="Available quotes"
                ondragover={(event) => event.preventDefault()}
                ondrop={(event) => dropQuote(event, false)}
                class="min-h-72 rounded-3xl border border-slate-200 bg-white p-5 shadow-sm"
            >

                <div class="mb-5 flex items-center justify-between">

                    <h2 class="text-xl font-bold">
                        Explore
                    </h2>

                    <span class="rounded-full bg-slate-100 px-3 py-1 text-sm text-slate-600">
                        {quotes.length - savedIds.length} quotes
                    </span>

                </div>

                <div class="space-y-4">

                    {#each quotes.filter((quote) => !savedIds.includes(quote.id)) as quote (quote.id)}

                        <div
                            role="group"
                            draggable="true"
                            ondragstart={(event) => dragStart(event, quote.id)}
                            class="cursor-grab active:cursor-grabbing"
                        >

                            <Quote
                                text={quote.text}
                                author={quote.author}
                                saved={false}
                                onMove={() => moveQuote(quote.id, true)}
                            />

                        </div>

                    {/each}

                </div>

                {#if savedIds.length === quotes.length}

                    <p class="py-12 text-center text-sm text-slate-400">
                        All quotes are in your collection.
                    </p>

                {/if}

            </section>


            <!-- RIGHT: Collection -->
            <section
                aria-label="Saved quotes"
                ondragover={(event) => event.preventDefault()}
                ondrop={(event) => dropQuote(event, true)}
                class="min-h-72 rounded-3xl border-2 border-dashed border-indigo-200 bg-indigo-50/60 p-5"
            >

                <div class="mb-5 flex items-center justify-between">

                    <h2 class="text-xl font-bold">
                        My collection ♡
                    </h2>

                    <span class="rounded-full bg-indigo-100 px-3 py-1 text-sm text-indigo-700">
                        {savedIds.length} saved
                    </span>

                </div>

                <div class="space-y-4">

                    {#each quotes.filter((quote) => savedIds.includes(quote.id)) as quote (quote.id)}

                        <div
                            role="group"
                            draggable="true"
                            ondragstart={(event) => dragStart(event, quote.id)}
                            class="cursor-grab active:cursor-grabbing"
                        >

                            <Quote
                                text={quote.text}
                                author={quote.author}
                                saved={true}
                                onMove={() => moveQuote(quote.id, false)}
                            />

                        </div>

                    {/each}

                </div>

                {#if savedIds.length === 0}

                    <p class="rounded-2xl border border-dashed border-indigo-200 px-4 py-14 text-center text-sm text-indigo-500">
                        Drop a quote here ✨
                    </p>

                {/if}

            </section>

        </div>


        <!-- Easter Egg -->
        <section class="mt-8 rounded-3xl bg-slate-900 p-6 text-white sm:p-8">

            <p class="text-xs font-bold uppercase tracking-[0.2em] text-indigo-300">
                ✦ Hidden quote
            </p>

            <h2 class="mt-2 text-2xl font-semibold">
                A little secret is waiting...
            </h2>

            <p class="mt-2 text-sm text-slate-300">
                Find the hidden location (within 100 m) to unlock a bonus quote.
            </p>

            {#if secretUnlocked}

                <div class="mt-5 rounded-2xl bg-white p-5 text-slate-900">

                    <p class="text-xs font-bold uppercase tracking-widest text-indigo-600">
                        Secret unlocked ✓
                    </p>

                    <blockquote class="mt-3 text-2xl font-semibold">
                        “The best discoveries begin when you choose to explore.”
                    </blockquote>

                    <p class="mt-3 text-sm text-slate-500">
                        — The Quote Archive
                    </p>

                </div>

            {:else}

                <button
                    type="button"
                    onclick={checkLocation}
                    class="mt-5 rounded-xl bg-indigo-400 px-5 py-3 font-semibold text-slate-950 hover:bg-indigo-300"
                >
                    {tracking ? 'Stop checking location' : 'Find the secret quote'}
                </button>

            {/if}

            <p role="status" class="mt-4 text-sm text-slate-300">
                {locationMessage}
            </p>

        </section>

    </div>

</main>