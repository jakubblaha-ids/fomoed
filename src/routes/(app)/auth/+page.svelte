<script lang="ts">
	import { writable } from 'svelte/store';
	import type { ActionData } from './$types';
	import { failure, success } from '$lib/utils';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import { auth_user } from '$lib/stores/user';
	import LoginForm from '$lib/comps/forms/LoginForm.svelte';
	import RegisterForm from '$lib/comps/forms/RegisterForm.svelte';
	import UpdatePasswordForm from '$lib/comps/forms/UpdatePasswordForm.svelte';
	import ForgotPasswordForm from '$lib/comps/forms/ForgotPasswordForm.svelte';
	import TwoPaneLayout from './TwoPaneLayout.svelte';

	const action = writable<'login' | 'signup' | 'forgot' | 'update_password'>('login');
	const uid = $page.url.searchParams.get('uid');

	onMount(() => {
		if ($page.url.searchParams.get('uid') && $auth_user) {
			action.set('update_password');
		}
	});

	export let form: ActionData;

	$: form && console.log('Auth Response: ', form);

	// TODO Change from toast notifications as they are not working as expected
	$: if (form) {
		if (form.error) {
			failure(form.error);
		} else {
			// if (form.action === 'signup') {
			// 	onMount(() => {
			// 		goto('/auth/sent/create-account?email=');
			// 	});
			// 	// success(
			// 	// 	'Successfully created your account. Please check your email for additional instructions'
			// 	// );
			// }
			// if (form.action === 'forgot') {
			// 	onMount(() => {
			// 		goto('/auth/sent/password-reset?email=' + form.email);
			// 	});
			// 	// success(
			// 	// 	'Successfully sent password reset email. Please check your email for the confirmation link'
			// 	// );
			// }
			if (form.action === 'update_password') {
				onMount(() => {
					goto('/').then(() => {
						success('Successfully updated password');
					});
				});
			}
		}
	}

	auth_user.subscribe((u) => {
		if ($page.url.searchParams.get('uid')) {
			action.set('update_password');
		}
	});
</script>

<TwoPaneLayout>
	<form method="POST" action="?/{$action + (uid ? '&uid=' + uid : '')}">
		{#if $action === 'signup'}
			<RegisterForm on:click-login-link={() => action.set('login')} />
		{/if}

		{#if $action === 'update_password'}
			<UpdatePasswordForm />
		{/if}

		{#if $action === 'forgot'}
			<ForgotPasswordForm on:click-login-link={() => action.set('login')} />
		{/if}

		{#if $action === 'login'}
			<LoginForm
				on:click-forgot-password={() => action.set('forgot')}
				on:click-register-link={() => action.set('signup')}
			/>
		{/if}
	</form>
</TwoPaneLayout>
