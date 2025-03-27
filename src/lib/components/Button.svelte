<script lang="ts">
	import type { Snippet } from 'svelte';

	interface ButtonProps {
		children: Snippet;
		type?: 'button' | 'submit';
		variant?: 'primary' | 'secondary';
		onclick?: (e: MouseEvent) => void;
		disabled?: boolean;
	}

	let { children, onclick, type, variant = 'primary', disabled, ...props }: ButtonProps = $props();
</script>

<button {...props} {onclick} {type} class={`btn btn--${variant} ${disabled ? 'btn-disabled' : ''}`}
	>{@render children()}</button
>

<style lang="scss">
	.btn {
		display: flex;
		cursor: pointer;
		align-items: center;
		transition: all 0.5s ease-in-out;
		gap: 0.5rem;
		outline: none;
		padding: 0.5rem 1rem;
		border-radius: 12px;

		&--primary {
			background-color: var(--primary);
			border: 1px solid var(--primary);
			color: var(--secondary);
		}

		&--secondary {
			background-color: var(--secondary);
			border: 1px solid var(--primary);
			color: var(--primary);
		}

		&:hover {
			border-radius: 10px;
		}
	}
</style>
