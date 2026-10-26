<script>
    let { author, number, children } = $props();
    let quoteElement;
    let toast = $state('');
    let timer;

    function getQuote() {
        return `“${quoteElement.innerText.trim()}”\n— ${author}`;
    }

    function showToast(text) {
        toast = text;
        clearTimeout(timer);
        timer = setTimeout(() => (toast = ''), 3000);
    }

    // Fallback for older browsers and devices without the Clipboard API.
    function oldCopy(text) {
        const input = document.createElement('textarea');
        input.value = text;
        input.style.position = 'fixed';
        input.style.opacity = '0';
        document.body.appendChild(input);

        input.focus();
        input.select();
        input.setSelectionRange(0, text.length);

        const copied = document.execCommand('copy');
        input.remove();

        if (!copied) throw new Error('Copy failed');
    }

    async function copyQuote() {
        const text = getQuote();

        try {
            if (navigator.clipboard?.writeText) {
                try {
                    await navigator.clipboard.writeText(text);
                } catch {
                    oldCopy(text);
                }
            } else {
                oldCopy(text);
            }

            showToast('Quote copied to clipboard!');
        } catch {
            showToast('Could not copy the quote.');
        }
    }

    async function shareQuote() {
        if (navigator.share) {
            try {
                await navigator.share({ text: getQuote() });
                showToast('Quote shared!');
            } catch (error) {
                if (error.name !== 'AbortError') {
                    await copyQuote();
                }
            }
        } else {
            await copyQuote();
        }
    }
</script>

<article class="flex min-h-80 flex-col rounded-3xl border border-white/10 bg-white/5 p-7 transition duration-200 hover:-translate-y-1 hover:border-violet-300/40 hover:bg-white/10">

    <div class="mb-5 flex items-start justify-between">
        <span class="rounded-full bg-violet-400/15 px-3 py-1 text-xs font-semibold tracking-widest text-violet-200">
            {number} / 05
        </span>

        <span aria-hidden="true" class="font-serif text-6xl leading-none text-violet-300">“</span>
    </div>

    <blockquote bind:this={quoteElement} class="flex-1 font-serif text-2xl leading-snug text-white">
        {@render children?.()}
    </blockquote>

    <p class="mt-5 text-sm font-medium text-violet-200">— {author}</p>

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

<!-- Notification -->
{#if toast}
    <div
        role="status"
        aria-live="polite"
        class="fixed bottom-6 left-1/2 z-50 flex w-max max-w-[90vw] -translate-x-1/2 items-center gap-3 rounded-2xl border border-violet-300/30 bg-slate-900 px-5 py-4 text-sm font-medium text-white shadow-2xl shadow-black/50"
    >
        <span class="flex h-7 w-7 shrink-0 items-center justify-center rounded-full bg-violet-300 font-bold text-slate-950">
            {toast.startsWith('Could not') ? '!' : '✓'}
        </span>

        {toast}
    </div>
{/if}