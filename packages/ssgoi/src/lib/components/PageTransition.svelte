<script lang="ts">
	import context from '$lib/context/index.js';
	import { none } from '$lib/transitions/index.js';
	import type { TransitionConfig } from '$lib/types.js';

	interface Props {
		class?: string;
		children?: import('svelte').Snippet;
	}

	let { class: className = '', children }: Props = $props();

	const config = context.config.get();
	const scrollHistoryContext = context.scrollHistory.get();
	const pageTransitionContext = context.pageTransition.get();
	const getFromScrollTop = () =>
		(pageTransitionContext.from == null
			? undefined
			: (scrollHistoryContext[pageTransitionContext.from.url.pathname] ?? 0)) ?? 0;
	const getToScrollTop = () =>
		(pageTransitionContext.to == null
			? undefined
			: scrollHistoryContext[pageTransitionContext.to.url.pathname]) ?? 0;

	function transitionIn(
		node: HTMLElement,
		params: { getFromScrollTop: () => number; getToScrollTop: () => number }
	): TransitionConfig {
		const { from, to } = pageTransitionContext;
		if (from == null || to == null) return none().in(node, params);

		const transition = config({ path: from.url.pathname }, { path: to.url.pathname });
		return transition.in(node, params);
	}

	function transitionOut(
		node: HTMLElement,
		params: { getFromScrollTop: () => number; getToScrollTop: () => number }
	): TransitionConfig {
		const { from, to } = pageTransitionContext;
		if (from == null || to == null) return none().out(node, params);

		const transition = config({ path: from.url.pathname }, { path: to.url.pathname });
		return transition.out(node, params);
	}
</script>

<div
	data-page-transition
	class={className}
	in:transitionIn|global={{ getFromScrollTop: getFromScrollTop, getToScrollTop: getToScrollTop }}
	out:transitionOut|global={{ getFromScrollTop: getFromScrollTop, getToScrollTop: getToScrollTop }}
>
	{@render children?.()}
</div>
