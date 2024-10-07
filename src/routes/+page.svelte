<script lang="ts">
	import OTPInput from '$lib/components/input.svelte';
	import OTPGroup from '$lib/components/group.svelte';
	import OTPSlot from '$lib/components/slot.svelte';

	let value = $state('');
</script>

<OTPInput bind:value maxLength={6}>
	<OTPGroup>
		<OTPSlot index={[0, 1, 2]} />
	</OTPGroup>
	<OTPGroup>
		<OTPSlot index={3} />
		<OTPSlot index={4} />
		<OTPSlot index={5} />
	</OTPGroup>
</OTPInput>
