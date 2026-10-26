<script>
    let { text, author, saved, onMove } = $props();

    let status = $state('');

    function quoteText() {
        return `“${text}”\n— ${author}`;
    }

    // Copy fallback for older browsers
    function oldCopy(value) {
        const input = document.createElement('textarea');

        input.value = value;
        input.style.position = 'fixed';
        input.style.opacity = '0';

        document.body.appendChild(input);

        input.focus();
        input.select();
        input.setSelectionRange(0, value.length);

        const copied = document.execCommand('copy');

        input.remove();

        if (!copied) {
            throw new Error('Copy failed');
        }
    }

    // Clipboard and Notifications API
    async function copyQuote() {
        const permission =
            'Notification' in window
                ? Notification.permission === 'default'
                    ? Notification.requestPermission()
                    : Promise.resolve(Notification.permission)
                : Promise.resolve('unsupported');

        try {
            if (navigator.clipboard?.writeText) {
                try {
                    await navigator.clipboard.writeText(quoteText());
                } catch {
                    oldCopy(quoteText());
                }
            } else {
                oldCopy(quoteText());
            }

            status = 'Copied!';
        } catch {
            status = 'Could not copy the quote.';
            return;
        }

        try {
            if ((await permission) !== 'granted') {
                return;
            }

            if ('serviceWorker' in navigator) {
                await navigator.serviceWorker.register('/sw.js');

                const registration = await navigator.serviceWorker.ready;

                await registration.showNotification('The Quote Archive', {
                    body: 'Quote copied to clipboard!'
                });
            } else {
                new Notification('The Quote Archive', {
                    body: 'Quote copied to clipboard!'
                });
            }
        } catch {
            // Copy still works if system notifications are unavailable.
        }
    }

    // Web Share API
    async function shareQuote() {
        if (!navigator.share) {
            await copyQuote();
            return;
        }

        try {
            await navigator.share({
                text: quoteText()
            });
        } catch (error) {
            if (error.name !== 'AbortError') {
                await copyQuote();
            }
        }
    }
</script>

<article class="rounded-2xl border border-slate-200 bg-white p-5 shadow-sm">

    <span
        class="text-3xl leading-none text-indigo-400"
        aria-hidden="true"
    >
        “
    </span>

    <blockquote class="mt-1 text-xl font-medium leading-relaxed text-slate-800">
        {text}
    </blockquote>

    <p class="mt-3 text-sm text-slate-500">
        — {author}
    </p>

    <div class="mt-5 flex flex-wrap items-center gap-2 border-t border-slate-100 pt-4">

        <!-- Copy -->
        <button
            type="button"
            onclick={copyQuote}
            class="rounded-lg bg-indigo-600 px-4 py-2 text-sm font-semibold text-white hover:bg-indigo-700"
        >
            Copy
        </button>

        <!-- Share -->
        <button
            type="button"
            onclick={shareQuote}
            class="rounded-lg border border-slate-200 px-4 py-2 text-sm font-semibold hover:bg-slate-50"
        >
            Share ↗
        </button>

        <!-- Move between columns -->
        <button
            type="button"
            onclick={onMove}
            class="ml-auto rounded-lg px-3 py-2 text-sm font-semibold text-indigo-700 hover:bg-indigo-50"
        >
            {saved ? '← Return' : 'Save →'}
        </button>

    </div>

    {#if status}
        <p role="status" class="mt-2 text-xs text-indigo-600">
            {status}
        </p>
    {/if}

</article>