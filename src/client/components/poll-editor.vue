<template>
<div class="zmdxowus">
	<ul ref="choices">
		<li v-for="(choice, i) in choices" :key="i">
			<MkInput class="input" :value="choice" @update:value="onInput(i, $event)">
				<span>{{ $t('_poll.choiceN', { n: i + 1 }) }}</span>
			</MkInput>
			<button v-if="choices.length > 2" @click="remove(i)" class="_button">
				<Fa :icon="faTimes"/>
			</button>
		</li>
	</ul>
	<MkButton class="add" v-if="choices.length < 10" @click="add"><fa :icon="faPlus" /></MkButton>
	<section>
		<MkSwitch v-model:value="multiple">{{ $ts._poll.canMultipleVote }}</MkSwitch>
		<div>
			<MkSelect v-model:value="expiration">
				<template #label>{{ $ts._poll.expiration }}</template>
				<option value="infinite">{{ $ts._poll.infinite }}</option>
				<option value="at">{{ $ts._poll.at }}</option>
				<option value="after">{{ $ts._poll.after }}</option>
			</MkSelect>
			<section class="_inputs" v-if="expiration === 'at'">
				<MkInput v-model:value="atDate" type="date" class="input">
					<span>{{ $ts._poll.deadlineDate }}</span>
				</MkInput>
				<MkInput v-model:value="atTime" type="time" class="input">
					<span>{{ $ts._poll.deadlineTime }}</span>
				</MkInput>
			</section>
			<section v-if="expiration === 'after'">
				<MkInput v-model:value="after" type="number" class="input">
					<span>{{ $ts._poll.duration }}</span>
				</MkInput>
				<MkSelect v-model:value="unit">
					<option value="second">{{ $ts._time.second }}</option>
					<option value="minute">{{ $ts._time.minute }}</option>
					<option value="hour">{{ $ts._time.hour }}</option>
					<option value="day">{{ $ts._time.day }}</option>
				</MkSelect>
			</section>
		</div>
	</section>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { faExclamationTriangle, faTimes, faPlus } from '@fortawesome/free-solid-svg-icons';
import { addTime } from '../../prelude/time';
import { formatDateTimeString } from '../../misc/format-time-string';
import MkInput from './ui/input.vue';
import MkSelect from './ui/select.vue';
import MkSwitch from './ui/switch.vue';
import MkButton from './ui/button.vue';

export default defineComponent({
	components: {
		MkInput,
		MkSelect,
		MkSwitch,
		MkButton,
	},

	props: {
		poll: {
			type: Object,
			required: true
		}
	},

	emits: ['updated'],

	data() {
		return {
			choices: this.poll.choices,
			multiple: this.poll.multiple,
			expiration: 'infinite',
			atDate: formatDateTimeString(addTime(new Date(), 1, 'day'), 'yyyy-MM-dd'),
			atTime: '00:00',
			after: 0,
			unit: 'second',
			faExclamationTriangle, faTimes, faPlus
		};
	},

	watch: {
		choices: {
			handler() {
				this.$emit('updated', this.get());
			},
			deep: true
		},
		multiple: {
			handler() {
				this.$emit('updated', this.get());
			},
		},
		expiration: {
			handler() {
				this.$emit('updated', this.get());
			},
		},
		atDate: {
			handler() {
				this.$emit('updated', this.get());
			},
		},
		after: {
			handler() {
				this.$emit('updated', this.get());
			},
		},
		unit: {
			handler() {
				this.$emit('updated', this.get());
			},
		},
	},

	created() {
		const poll = this.poll;
		if (poll.expiresAt) {
			this.expiration = 'at';
			this.atDate = this.atTime = poll.expiresAt;
		} else if (typeof poll.expiredAfter === 'number') {
			this.expiration = 'after';
			this.after = poll.expiredAfter / 1000;
		} else {
			this.expiration = 'infinite';
		}
	},

	methods: {
		onInput(i, e) {
			this.choices[i] = e;
		},

		add() {
			this.choices.push('');
			this.$nextTick(() => {
				// TODO
				//(this.$refs.choices as any).childNodes[this.choices.length - 1].childNodes[0].focus();
			});
		},

		remove(i) {
			this.choices = this.choices.filter((_, _i) => _i != i);
		},

		get() {
			const at = () => {
				return new Date(`${this.atDate} ${this.atTime}`).getTime();
			};

			const after = () => {
				let base = parseInt(this.after);
				switch (this.unit) {
					case 'day': base *= 24;
					case 'hour': base *= 60;
					case 'minute': base *= 60;
					case 'second': return base *= 1000;
					default: return null;
				}
			};

			return {
				choices: this.choices,
				multiple: this.multiple,
				...(
					this.expiration === 'at' ? { expiresAt: at() } :
					this.expiration === 'after' ? { expiredAfter: after() } : {}
				)
			};
		},
	}
});
</script>

<style lang="scss" scoped>
.zmdxowus {
	padding: 8px;

	> .caution {
		margin: 0 0 8px 0;
		font-size: 0.8em;
		color: #f00;

		> [data-icon] {
			margin-right: 4px;
		}
	}

	> ul {
		display: block;
		margin: 0;
		padding: 0;
		list-style: none;

		> li {
			display: flex;
			margin: 8px 0;
			padding: 0;
			width: 100%;

			> .input {
				flex: 1;
				margin-top: 16px;
				margin-bottom: 0;
			}

			> button {
				width: 32px;
				padding: 4px 0;
			}
		}
	}

	> .add {
		margin: 8px 0 0 0;
		width: 100%;
		height: 24px;
		padding: 0;
		z-index: 1;
	}

	> section {
		margin: 16px 0 -16px 0;

		> div {
			margin: 0 8px;

			&:last-child {
				flex: 1 0 auto;

				> section {
					align-items: center;
					display: flex;
					margin: -12px 0 8px 0;

					> &:first-child {
						margin-right: 16px;
					}

					> .input {
						flex: 1 0 auto;
					}
				}
			}
		}
	}
}
</style>
