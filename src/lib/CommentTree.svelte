<style lang="scss">
	.nested {
		border-left: 2px solid var(--sx-gray-300);
		border-bottom: 2px solid var(--sx-gray-300);
	}
	.comment-leaf:not(.nested) {
		border-bottom: 2px solid var(--sx-gray-300);
	}
	.comment:last-of-type {
		padding-bottom: var(--sx-spa);
	}
	.comment-leaf:hover {
		background: var(--sx-gray-transparent);
	}
	.collapsed {
		max-height: 4rem;
		overflow: hidden;
		opacity: 0.5;
	}
</style>

<Stack gap={0} cl="px-4">
	<!-- {#each commentTree as { cv, depth }} -->
	<VirtualFeed
		feedSize={commentTree.length}
		on:more
		{endOfFeed}
		{feedEndIcon}
		{feedEndMessage}
		loading={loadingContent}
		loadMoreFailed={loadingContentFailed}
	>
		<svelte:fragment let:index>
			{@const { cv, depth } = commentTree[index]}
			{@const collapsed = collapsedComments.some((c) => cv.comment.id === c)}
			<div class="comment">
				<div
					class="comment-leaf p-2 pb-0"
					style:margin-left="calc(var(--sx-spacing-6) * {depth})"
					class:collapsed
					class:nested={depth > 0}
				>
					<Comment
						commentView={cv}
						{postOP}
						on:collapse={() => toggleCollapse(cv.comment.id)}
						{collapsed}
						on:new-comment
						on:update-comment
					/>
					{#if cv.counts.child_count > 0 && !collapsed && loadedChildren(cv.comment.id) === 0}
						{@const loading = expandLoadingIds.includes(cv.comment.id)}
						<div>
							<button
								class="tertiary f-row gap-2"
								on:click={() => dispatch('expand', cv.comment.id)}
								disabled={loading}
							>
								{#if loading}
									<Spinner />
								{/if}
								Load {cv.counts.child_count}
								{cv.counts.child_count === 1 ? 'reply' : 'replies'}</button
							>
						</div>
					{/if}
				</div>
			</div>
		</svelte:fragment>
	</VirtualFeed>
</Stack>

<script lang="ts">
	import { Stack } from 'sheodox-ui';
	import Comment from './Comment.svelte';
	import VirtualFeed from './VirtualFeed.svelte';
	import type { CommentView } from 'lemmy-js-client';
	import { createEventDispatcher } from 'svelte';
	import Spinner from './Spinner.svelte';

	let collapsedCommentSet = new Set<number>(),
		collapsedComments: number[] = [];

	function toggleCollapse(commentId: number) {
		collapsedCommentSet.has(commentId) ? collapsedCommentSet.delete(commentId) : collapsedCommentSet.add(commentId);
		collapsedComments = Array.from(collapsedCommentSet);
	}

	const dispatch = createEventDispatcher<{
		expand: number;
		more: void;
	}>();

	interface CommentBranch {
		cv: CommentView;
		depth: number;
		path: string;
	}

	export let postOP: string;
	export let commentViews: CommentView[];
	export let path = '0';
	export let rootCommentId: number | null = null;
	export let loadingContent: boolean;
	export let loadingContentFailed: boolean;
	export let feedEndMessage: string;
	export let feedEndIcon: string;
	export let endOfFeed: boolean;
	export let expandLoadingIds: number[];

	$: rootPath = getRootPath(rootCommentId, path);

	function getRootPath(rootId: number | null, path: string) {
		if (rootCommentId === null) {
			return path;
		}

		const rootCV = commentViews.find((cv) => cv.comment.id === rootId);

		if (rootCV) {
			const rootPath = rootCV.comment.path.split('.');
			rootPath.pop();
			return rootPath.join('.');
		}

		return '0';
	}

	function loadedChildren(id: number) {
		let total = 0;
		for (let i = 0; i < commentViews.length; i++) {
			const commentPath = commentViews[i].comment.path;
			if (commentPath.includes(id + '') && !commentPath.endsWith('.' + id)) {
				total++;
			}
		}
		return total;
	}

	// each
	function getBranches(
		path: string,
		commentViews: CommentView[] = [],
		depth: number,
		collapsed: number[]
	): CommentBranch[] {
		return commentViews
			.filter((cv) => {
				return cv.comment.path === path + '.' + cv.comment.id && !collapsed.some((c) => path.includes('' + c));
			})
			.map((cv) => {
				return [{ cv, depth, path }, ...getBranches(path + '.' + cv.comment.id, commentViews, depth + 1, collapsed)];
			})
			.flat();
	}

	$: commentTree = getBranches(rootPath, commentViews, 0, Array.from(collapsedComments));
</script>
