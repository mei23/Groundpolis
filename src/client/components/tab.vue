<script lang="ts">
import { defineComponent, h, resolveDirective, withDirectives } from 'vue';

export default defineComponent({
	props: {
		value: {
			required: true,
		},
	},
	render() {
		const options = this.$slots.default();

		return withDirectives(h('div', {
			class: 'pxhvhrfw',
		}, options.map(option => h('button', {
			class: ['_button _clickable', { active: this.value === option.props.value }],
			key: option.props.value,
			disabled: this.value === option.props.value,
			onClick: () => {
				this.$emit('update:value', option.props.value);
			}
		}, option.children))), [
			[resolveDirective('size'), { max: [500] }]
		]);
	}
});
</script>

<style lang="scss">
.pxhvhrfw {
	display: flex;

	> button {
		flex: 1;
		padding: 15px 12px 12px 12px;
		border-bottom: solid 3px transparent;

		&:disabled {
			opacity: 1 !important;
			cursor: default;
		}

		&.active {
			color: var(--accent);
			border-bottom-color: var(--accent);
		}

		&:not(.active):hover {
			color: var(--fgHighlighted);
		}

		> .icon {
			margin-right: 6px;
		}
	}

	&.max-width_500px {
		font-size: 80%;

		> button {
			padding: 11px 8px 8px 8px;
		}
	}
}
</style>
