<template>
<div class="qglefbjs" :class="[ notification.type, { compact: isCompactMode } ]" v-size="{ max: [500, 600] }">
	<div class="head" v-if="notification.type !== 'system'">
		<MkAvatar v-if="notification.user" class="icon" :user="notification.user"/>
		<img v-else-if="notification.icon" class="icon" :src="notification.icon" alt=""/>
		<div class="sub-icon" :class="notification.type !== 'reaction' || !$store.reactiveState.disableReactions.value ? notification.type : 'like'">
			<Fa :icon="faPlus" v-if="notification.type === 'follow'"/>
			<Fa :icon="faClock" v-else-if="notification.type === 'receiveFollowRequest'"/>
			<Fa :icon="faCheck" v-else-if="notification.type === 'followRequestAccepted'"/>
			<Fa :icon="faIdCardAlt" v-else-if="notification.type === 'groupInvited'"/>
			<Fa :icon="faRetweet" v-else-if="notification.type === 'renote'"/>
			<Fa :icon="faReply" v-else-if="notification.type === 'reply'"/>
			<Fa :icon="faAt" v-else-if="notification.type === 'mention'"/>
			<Fa :icon="faQuoteLeft" v-else-if="notification.type === 'quote'"/>
			<Fa :icon="faPollH" v-else-if="notification.type === 'pollVote'"/>
			<template v-else-if="notification.type === 'reaction'">
				<XReactionIcon v-if="!$store.reactiveState.disableReactions.value" :reaction="notification.reaction" :custom-emojis="notification.note.emojis" :no-style="true"/>
				<Fa v-else :icon="faHeart"/>
			</template>
		</div>
	</div>
	<div class="tail">
		<header>
			<MkA v-if="notification.user" class="name" :to="userPage(notification.user)" v-user-preview="notification.user.id"><MkUserName :user="notification.user"/></MkA>
			<span v-else style="font-weight: bold">{{ notification.header }}</span>
			<MkTime :time="notification.createdAt" v-if="withTime" class="time"/>
		</header>
		<MkA v-if="notification.type === 'reaction'" class="text" :to="notePage(notification.note)" :title="getNoteSummary(notification.note)">
			<Fa :icon="faQuoteLeft"/>
			<Mfm :text="getNoteSummary(notification.note)" :plain="true" :nowrap="!full" :custom-emojis="notification.note.emojis"/>
			<Fa :icon="faQuoteRight"/>
		</MkA>
		<MkA v-if="notification.type === 'renote'" class="text" :to="notePage(notification.note)" :title="getNoteSummary(notification.note.renote)">
			<Fa :icon="faQuoteLeft"/>
			<Mfm :text="getNoteSummary(notification.note.renote)" :plain="true" :nowrap="!full" :custom-emojis="notification.note.renote.emojis"/>
			<Fa :icon="faQuoteRight"/>
		</MkA>
		<MkA v-if="notification.type === 'reply'" class="text" :to="notePage(notification.note)" :title="getNoteSummary(notification.note)">
			<Mfm :text="getNoteSummary(notification.note)" :plain="true" :nowrap="!full" :custom-emojis="notification.note.emojis"/>
		</MkA>
		<MkA v-if="notification.type === 'mention'" class="text" :to="notePage(notification.note)" :title="getNoteSummary(notification.note)">
			<Mfm :text="getNoteSummary(notification.note)" :plain="true" :nowrap="!full" :custom-emojis="notification.note.emojis"/>
		</MkA>
		<MkA v-if="notification.type === 'quote'" class="text" :to="notePage(notification.note)" :title="getNoteSummary(notification.note)">
			<Mfm :text="getNoteSummary(notification.note)" :plain="true" :nowrap="!full" :custom-emojis="notification.note.emojis"/>
		</MkA>
		<MkA v-if="notification.type === 'pollVote'" class="text" :to="notePage(notification.note)" :title="getNoteSummary(notification.note)">
			<Fa :icon="faQuoteLeft"/>
			<Mfm :text="getNoteSummary(notification.note)" :plain="true" :nowrap="!full" :custom-emojis="notification.note.emojis"/>
			<Fa :icon="faQuoteRight"/>
		</MkA>
		<span v-if="notification.type === 'follow'" class="text" style="opacity: 0.6;">
			{{ $ts.youGotNewFollower }}
			<div v-if="full">
				<MkFollowButton :user="notification.user" full disableIfFollowing/>
			</div>
		</span>
		<span v-if="notification.type === 'followRequestAccepted'" class="text" style="opacity: 0.6;">{{ $ts.followRequestAccepted }}</span>
		<span v-if="notification.type === 'receiveFollowRequest'" class="text" style="opacity: 0.6;">{{ $ts.receiveFollowRequest }}
			<div v-if="full && !followRequestDone">
				<button class="_textButton" @click="acceptFollowRequest()">{{ $ts.accept }}</button>
				|
				<button class="_textButton" @click="rejectFollowRequest()">{{ $ts.reject }}</button>
			</div>
		</span>
		<span v-if="notification.type === 'groupInvited'" class="text" style="opacity: 0.6;">
			{{ $ts.groupInvited }}: <b>{{ notification.invitation.group.name }}</b>
			<div v-if="full && !groupInviteDone">
				<button class="_textButton" @click="acceptGroupInvitation()">{{ $ts.accept }}</button>
				|
				<button class="_textButton" @click="rejectGroupInvitation()">{{ $ts.reject }}</button>
			</div>
		</span>
		<span v-if="notification.type === 'app' || notification.type === 'system'" class="text">
			<Mfm :text="notification.body" :nowrap="!full"/>
		</span>
	</div>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { faIdCardAlt, faPlus, faQuoteLeft, faQuoteRight, faRetweet, faReply, faAt, faCheck, faPollH, faHeart } from '@fortawesome/free-solid-svg-icons';
import { faClock } from '@fortawesome/free-regular-svg-icons';
import { getNoteSummary } from '../../misc/get-note-summary';
import XReactionIcon from './reaction-icon.vue';
import MkFollowButton from './follow-button.vue';
import notePage from '../filters/note';
import { userPage } from '../filters/user';
import { i18n } from '@/i18n';
import * as os from '@/os';

const locale = i18n.locale;

export default defineComponent({
	components: {
		XReactionIcon, MkFollowButton
	},
	props: {
		notification: {
			type: Object,
			required: true,
		},
		withTime: {
			type: Boolean,
			required: false,
			default: false,
		},
		full: {
			type: Boolean,
			required: false,
			default: false,
		},
	},

	data() {
		return {
			getNoteSummary: (text: string) => getNoteSummary(text, i18n.locale),
			followRequestDone: false,
			groupInviteDone: false,
			connection: null,
			readObserver: null,
			faIdCardAlt, faPlus, faQuoteLeft, faQuoteRight, faRetweet, faReply, faAt, faClock, faCheck, faPollH, faHeart
		};
	},

	computed: {
		isCompactMode(): boolean {
			return this.$store.state.postStyle === 'compact';
		},
	},

	mounted() {
		if (this.notification.isRead) return;
		if (this.notification.type === 'system') return;

		this.readObserver = new IntersectionObserver((entries, observer) => {
			if (!entries.some(entry => entry.isIntersecting)) return;
			os.stream.send('readNotification', {
				id: this.notification.id
			});
			entries.map(({ target }) => observer.unobserve(target));
		});

		this.readObserver.observe(this.$el);

		this.connection = os.stream.useSharedConnection('main');
		this.connection.on('readAllNotifications', () => this.readObserver.unobserve(this.$el));
	},

	beforeUnmount() {
		if (this.notification.isRead) return;
		if (this.notification.type === 'system') return;

		this.readObserver.unobserve(this.$el);
		this.connection.dispose();
	},

	methods: {
		acceptFollowRequest() {
			this.followRequestDone = true;
			os.api('following/requests/accept', { userId: this.notification.user.id });
		},
		rejectFollowRequest() {
			this.followRequestDone = true;
			os.api('following/requests/reject', { userId: this.notification.user.id });
		},
		acceptGroupInvitation() {
			this.groupInviteDone = true;
			os.apiWithDialog('users/groups/invitations/accept', { invitationId: this.notification.invitation.id });
		},
		rejectGroupInvitation() {
			this.groupInviteDone = true;
			os.api('users/groups/invitations/reject', { invitationId: this.notification.invitation.id });
		},
		notePage,
		userPage
	}
});
</script>

<style lang="scss" scoped>
.qglefbjs {
	position: relative;
	box-sizing: border-box;
	padding: 24px 32px;
	font-size: 0.9em;
	overflow-wrap: break-word;
	display: flex;
	contain: content;

	&.compact {
		padding: 8px;
		> .head {
			position: relative;
			width: unset;

			> .icon {
				width: 1.5em;
				height: 1.5em;
				display: inline-block;
			}

			> .sub-icon {
				position: relative;
				box-shadow: none;
				display: inline-block;
			}
		}
	}

	&.max-width_600px {
		padding: 16px;
		font-size: 0.9em;
	}

	&.max-width_500px {
		padding: 12px;
		font-size: 0.8em;
	}

	&:after {
		content: "";
		display: block;
		clear: both;
	}

	> .head {
		position: sticky;
		top: 0;
		flex-shrink: 0;
		width: 42px;
		height: 42px;
		margin-right: 8px;

		> .icon {
			display: block;
			width: 100%;
			height: 100%;
			// border-radius: 6px;
		}

		> .sub-icon {
			position: absolute;
			z-index: 1;
			bottom: -2px;
			right: -2px;
			width: 20px;
			height: 20px;
			box-sizing: border-box;
			border-radius: 100%;
			background: var(--panel);
			box-shadow: 0 0 0 3px var(--panel);
			font-size: 12px;
			pointer-events: none;

			&:empty {
				display: none;
			}

			> * {
				color: #fff;
				width: 100%;
				height: 100%;
			}

			&.follow, &.followRequestAccepted, &.receiveFollowRequest, &.groupInvited {
				padding: 3px;
				background: #36aed2;
			}

			&.renote {
				padding: 3px;
				background: #36d298;
			}

			&.quote {
				padding: 3px;
				background: #36d298;
			}

			&.reply {
				padding: 3px;
				background: #007aff;
			}

			&.mention {
				padding: 3px;
				background: #88a6b7;
			}

			&.pollVote {
				padding: 3px;
				background: #88a6b7;
			}

			&.like {
				padding: 3px;
				background: #f06292;
			}
		}
	}

	> .tail {
		flex: 1;
		min-width: 0;

		> header {
			display: flex;
			align-items: baseline;
			white-space: nowrap;

			> .name {
				text-overflow: ellipsis;
				white-space: nowrap;
				min-width: 0;
				overflow: hidden;
			}

			> .time {
				margin-left: auto;
				font-size: 0.9em;
			}
		}

		> .text {
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;

			> [data-icon] {
				vertical-align: super;
				font-size: 50%;
				opacity: 0.5;
			}

			> [data-icon]:first-child {
				margin-right: 4px;
			}

			> [data-icon]:last-child {
				margin-left: 4px;
			}
		}
	}
}
</style>
