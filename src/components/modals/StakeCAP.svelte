<script>
	
	import Modal from './Modal.svelte'
	import Input from '@components/layout/Input.svelte'
	import Button from '@components/layout/Button.svelte'
	
	import { onMount } from 'svelte'

	import { depositCAP, getCAPWalletBalance } from '@api/cap'
	import { approveAsset, getAllowance } from '@api/assets'
	import { formatCAPForDisplay } from "@lib/formatters"
	import { allowances } from '@lib/stores'
	import { focusInput, hideModal } from '@lib/ui'
	import LabelValue from '../layout/LabelValue.svelte'

	let amount, isSubmitting, isApproving = false, isCheckingAllowance = true, walletBalance = "0.0";

	$: formattedWalletBalance = formatCAPForDisplay(walletBalance);
	$: currentAllowance = $allowances['CAP']?.['Staking'];
	$: needsApproval = currentAllowance === undefined || currentAllowance * 1 < (amount * 1 || 0);

	async function submit() {
		if (!amount) return focusInput('Amount');
		isSubmitting = true;
		try {
			const success = await depositCAP(amount);
			if (success) hideModal();
		} finally {
			isSubmitting = false;
		}
	}

	async function checkAllowance() {
		isCheckingAllowance = true;
		try {
			await getAllowance('CAP', 'Staking');
		} finally {
			isCheckingAllowance = false;
		}
	}

	async function _approveAsset() {
		if (isApproving || isCheckingAllowance) return;
		isApproving = true;
		try {
			await approveAsset('CAP', 'Staking');
			await getAllowance('CAP', 'Staking');
		} finally {
			isApproving = false;
		}
	}

	async function getBalance() {
		walletBalance = await getCAPWalletBalance();
	}

	checkAllowance();
	getBalance();

	onMount(() => {
		focusInput('Amount');
	});

</script>

<Modal title='Stake CAP' width={280}>
	
	<div class='container'>

		<form on:submit|preventDefault={submit}>

			<div class="group">
				<Input label='Amount' bind:value={amount} />
				<LabelValue
					label="Wallet Balance"
					value={formattedWalletBalance}
					isClickable={true}
					hasSemiPadding={true}
					on:click={() => { amount = formattedWalletBalance; }}
				/>
			</div>

			<div>
				{#if isCheckingAllowance || isApproving}
					<Button noSubmit={true} isLoading={true} label='Checking approval' />
				{:else if needsApproval}
					<Button noSubmit={true} isLoading={isApproving} label='Approve CAP' on:click={_approveAsset} />
				{:else}
					<Button isLoading={isSubmitting} label='Stake' />
				{/if}
			</div>
		</form>

	</div>

</Modal>
