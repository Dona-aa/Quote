<script>
    let { author, number, children } = $props();
    let quoteElement;

    function getQuote() {
        return `“${quoteElement.innerText.trim()}”\n— ${author}`;
    }

    // Ask for notification permission
    function notificationPermission() {
        if (!('Notification' in window)) {
            return Promise.resolve('unsupported');
        }

        if (Notification.permission === 'default') {
            return Notification.requestPermission();
        }

        return Promise.resolve(Notification.permission);
    }

    // Show a real system notification
    async function notify(message, permission) {
        if ((await permission) !== 'granted') return;

        try {
            if ('serviceWorker' in navigator) {
                await navigator.serviceWorker.register('/sw.js');

                const worker = await navigator.serviceWorker.ready;

                await worker.showNotification('The Quote Archive', {
                    body: message
                });
            } else {
                new Notification('The Quote Archive', {
                    body: message
                });
            }
        } catch (error) {
            console.error('Notification failed:', error);
        }
    }

    // Copy fallback for older browsers
    function oldCopy(text) {
        const input = document.createElement('textarea');

        input.value = text;
        input.style.position = 'fixed';
        input.style.opacity = '0';

        document.body.appendChild(input);

        input.focus();
        input.select();

        const copied = document.execCommand('copy');

        input.remove();

        if (!copied) throw new Error('Copy failed');
    }

    // Copy quote
    async function copyQuote() {
        const permission = notificationPermission();

        try {
            if (navigator.clipboard?.writeText) {
                try {
                    await navigator.clipboard.writeText(getQuote());
                } catch {
                    oldCopy(getQuote());
                }
            } else {
                oldCopy(getQuote());
            }

            await notify('Quote copied to clipboard!', permission);
        } catch {
            await notify('Could not copy the quote.', permission);
        }
    }

    // Share quote
    async function shareQuote() {
        if (!navigator.share) {
            await copyQuote();
            return;
        }

        try {
            await navigator.share({
                text: getQuote()
            });
        } catch (error) {
            if (error.name !== 'AbortError') {
                await copyQuote();
            }
        }
    }
</script>

<article class="flex min-h-80 flex-col rounded-3xl border border-white/10 bg-white/5 p-7 transition duration-200 hover:-translate-y-1 hover:border-violet-300/40 hover:bg-white/10">

    <div class="mb-5 flex items-start justify-between">

        <span class="rounded-full bg-violet-400/15 px-3 py-1 text-xs font-semibold tracking-widest text-violet-200">
            {number} / 05
        </span>

        <span aria-hidden="true" class="font-serif text-6xl leading-none text-violet-300">
            “
        </span>

    </div>

    <blockquote
        bind:this={quoteElement}
        class="flex-1 font-serif text-2xl leading-snug text-white"
    >
        {@render children?.()}
    </blockquote>

    <p class="mt-5 text-sm font-medium text-violet-200">
        — {author}
    </p>

    <div class="mt-6 flex gap-3 border-t border-white/10 pt-5">

        <button
            onclick={copyQuote}
            class="flex-1 rounded-xl bg-violet-300 px-4 py-2.5 text-sm font-semibold text-slate-950 transition hover:bg-violet-200"
        >
            Copy
        </button>

        <button
            onclick={shareQuote}
            class="flex-1 rounded-xl border border-white/20 px-4 py-2.5 text-sm font-semibold text-white transition hover:bg-white/10"
        >
            Share
        </button>

    </div>

</article>