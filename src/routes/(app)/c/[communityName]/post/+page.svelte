<Title title="Post" />

<Breadcrumbs {links} />
<h1>Post</h1>
<form method="POST" action="?/post" use:enhance={submitFn} on:submit={() => (submitting = true)}>
	<PostCompose errorMessage={form?.errMsg} {submitting} communityId={data.communityView.community.id} />
</form>

<script lang="ts">
	import { enhance } from '$app/forms';
	import PostCompose from '$lib/PostCompose.svelte';
	import { Breadcrumbs } from 'sheodox-ui';
	import Title from '$lib/Title.svelte';
	import type { SubmitFunction } from '@sveltejs/kit';

	export let data;
	export let form;

	let submitting = false;

	const submitFn: SubmitFunction = () => {
		return async ({ update }) => {
			await update();
			submitting = false;
		};
	};

	$: links = [
		{
			text: 'Home',
			href: '/'
		},
		{
			text: data.communityName,
			href: `/c/${data.communityName}/`
		},
		{
			text: 'Post'
		}
	];
</script>
