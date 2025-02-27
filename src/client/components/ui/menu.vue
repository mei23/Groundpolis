
<template>
<div class="rrevdjwt"
	ref="items"
	@contextmenu.self="e => e.preventDefault()"
	v-hotkey="keymap"
>
	<template v-for="(item, i) in _items">
		<div v-if="item === null" class="divider"></div>
		<span v-else-if="item.type === 'label'" class="label item">
			<span>{{ item.text }}</span>
		</span>
		<span v-else-if="item.type === 'pending'" :tabindex="i" class="pending item">
			<span><MkEllipsis/></span>
		</span>
		<MkA v-else-if="item.type === 'link'" :to="item.to" @click.passive="close()" :tabindex="i" class="_button item">
			<Fa v-if="item.icon" :icon="item.icon" fixed-width/>
			<MkAvatar v-if="item.avatar" :user="item.avatar" class="avatar"/>
			<span>{{ item.text }}</span>
			<i v-if="item.indicate"><Fa :icon="faCircle"/></i>
		</MkA>
		<a v-else-if="item.type === 'a'" :href="item.href" :target="item.target" :download="item.download" @click="close()" :tabindex="i" class="_button item">
			<Fa v-if="item.icon" :icon="item.icon" fixed-width/>
			<span>{{ item.text }}</span>
			<i v-if="item.indicate"><Fa :icon="faCircle"/></i>
		</a>
		<button v-else-if="item.type === 'user'" @click="clicked(item.action, $event)" :tabindex="i" class="_button item">
			<MkAvatar :user="item.user" class="avatar"/><MkUserName :user="item.user"/>
			<i v-if="item.indicate"><Fa :icon="faCircle"/></i>
		</button>
		<button v-else @click="clicked(item.action, $event)" :tabindex="i" class="_button item" :class="{ danger: item.danger }">
			<Fa v-if="item.icon" :icon="item.icon" fixed-width/>
			<MkAvatar v-if="item.avatar" :user="item.avatar" class="avatar"/>
			<span>{{ item.text }}</span>
			<i v-if="item.indicate"><Fa :icon="faCircle"/></i>
		</button>
	</template>
	<span v-if="_items.length === 0" class="none item">
		<span>{{ $ts.none }}</span>
	</span>
</div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { faCircle } from '@fortawesome/free-solid-svg-icons';
import { focusPrev, focusNext } from '@/scripts/focus';
import contains from '@/scripts/contains';

export default defineComponent({
	props: {
		items: {
			type: Array,
			required: true
		},
		viaKeyboard: {
			type: Boolean,
			required: false
		},
	},
	emits: ['close'],
	data() {
		return {
			_items: [] as unknown[],
			faCircle,
		};
	},
	computed: {
		keymap(): any {
			return {
				'up|k|shift+tab': this.focusUp,
				'down|j|tab': this.focusDown,
				'esc': this.close,
			};
		},
	},
	created() {
		const items = this.items.filter(item => item !== undefined);

		let _items = [ ...items ];

		const queue: { i: number, promise: Promise<any> }[] = [];

		for (let i = 0; i < items.length; i++) {
			const item = items[i];
			
			if (item && item.then) { 
				// もし Promise なら
				// 一旦 pending アンカーを作って置いておく
				items[i] = { type: 'pending' };

				queue.push({
					i, promise: item as Promise<any>,
				});
			}
		}

		this._items = items;

		Promise.all(queue.map(o => o.promise.then(v => ({ i: o.i, value: v }))))
			.then(q => {
				q.forEach(({ i, value }) => {
					_items[i] = value;
				});
				_items = _items.flat();
				this._items = _items;
			});
	},
	mounted() {
		if (this.viaKeyboard) {
			this.$nextTick(() => {
				focusNext(this.$refs.items.children[0], true, false);
			});
		}

		if (this.contextmenuEvent) {
			this.$el.style.top = this.contextmenuEvent.pageY + 'px';
			this.$el.style.left = this.contextmenuEvent.pageX + 'px';

			for (const el of Array.from(document.querySelectorAll('body *'))) {
				el.addEventListener('mousedown', this.onMousedown);
			}
		}
	},
	beforeUnmount() {
		for (const el of Array.from(document.querySelectorAll('body *'))) {
			el.removeEventListener('mousedown', this.onMousedown);
		}
	},
	methods: {
		clicked(fn, ev) {
			fn(ev);
			this.close();
		},
		close() {
			this.$emit('close');
		},
		focusUp() {
			focusPrev(document.activeElement);
		},
		focusDown() {
			focusNext(document.activeElement);
		},
		onMousedown(e) {
			if (!contains(this.$el, e.target) && (this.$el != e.target)) this.close();
		},
	}
});
</script>

<style lang="scss" scoped>
.rrevdjwt {
	padding: 8px 0;

	> .item {
		display: block;
		position: relative;
		padding: 8px 16px;
		width: 100%;
		box-sizing: border-box;
		white-space: nowrap;
		font-size: 0.9em;
		line-height: 20px;
		text-align: left;
		overflow: hidden;
		text-overflow: ellipsis;

		&.danger {
			color: #ff2a2a;

			&:hover {
				color: #fff;
				background: #ff4242;
			}

			&:active {
				color: #fff;
				background: #d42e2e;
			}
		}

		&:hover {
			color: #fff;
			background: var(--accent);
			text-decoration: none;
		}

		&:active {
			color: #fff;
			background: var(--accentDarken);
		}

		&:not(:active):focus {
			box-shadow: 0 0 0 2px var(--focus) inset;
		}

		&.label {
			pointer-events: none;
			font-size: 0.7em;
			padding-bottom: 4px;

			> span {
				opacity: 0.7;
			}
		}

		&.pending {
			pointer-events: none;
			opacity: 0.7;
		}

		&.none {
			pointer-events: none;
			opacity: 0.7;
		}

		> [data-icon] {
			margin-right: 12px;
			width: 20px;
		}

		> .avatar {
			margin-right: 4px;
			width: 20px;
			height: 20px;
		}

		> i {
			position: absolute;
			top: 5px;
			left: 13px;
			color: var(--indicator);
			font-size: 12px;
			animation: blink 1s infinite;
		}
	}

	> .divider {
		margin: 8px 0;
		height: 1px;
		background: var(--divider);
	}
}
</style>
