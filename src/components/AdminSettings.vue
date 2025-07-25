<template>
	<div id="slack_prefs" class="section">
		<h2>
			<SlackIcon class="icon" />
			{{ t('integration_slack', 'Slack integration') }}
		</h2>
		<p class="settings-hint">
			<span>
				{{ t('integration_slack', 'To allow your Nextcloud users to use OAuth to authenticate to the Slack app, and set the ID and secret here.') }}
				{{ " " }}
				<a target="_blank" href="https://api.slack.com/apps">{{ t('integration_slack', 'Link to create a Slack application') }} ↗</a>
			</span>
		</p>
		<br>
		<p class="settings-hint">
			<InformationOutlineIcon :size="24" class="icon" />
			{{ t('integration_slack', 'Make sure you set the "Redirect URI" in the "OAuth & Permissions" section of your Slack app settings to') }}
		</p>
		<strong>{{ redirect_uri }}</strong>
		<br><br>
		<p class="settings-hint">
			{{ t('integration_slack', 'Put the "Client ID" and "Client secret" below. Your Nextcloud users will then see a "Connect to Slack" button in their personal settings.') }}
		</p>
		<p class="settings-hint">
			{{ t('integration_slack', 'In the same "OAuth & Permissions" section, token rotation can be enabled. If enabled, your access token would be regularly refreshed with a refresh token. This is handled automatically.') }}
		</p>
		<br>
		<div id="slack-content">
			<div class="line">
				<label for="slack-client-id">
					<AccountOutlineIcon :size="20" class="icon" />
					{{ t('integration_slack', 'Client ID') }}
				</label>
				<input id="slack-client-id"
					v-model="state.client_id"
					type="password"
					:readonly="readonly"
					:placeholder="t('integration_slack', 'ID of your Slack application')"
					@input="onInput"
					@focus="readonly = false">
			</div>
			<div class="line">
				<label for="slack-client-secret">
					<KeyOutlineIcon :size="20" class="icon" />
					{{ t('integration_slack', 'Application secret') }}
				</label>
				<input id="slack-client-secret"
					v-model="state.client_secret"
					type="password"
					:readonly="readonly"
					:placeholder="t('integration_slack', 'Client secret of your Slack application')"
					@focus="readonly = false"
					@input="onInput">
			</div>
			<div class="line slack-checkbox">
				<NcCheckboxRadioSwitch
					:checked.sync="state.use_popup"
					@update:checked="onUsePopupChanged">
					{{ t('integration_slack', 'Use a popup to authenticate') }}
				</NcCheckboxRadioSwitch>
			</div>
		</div>
	</div>
</template>

<script>
import AccountOutlineIcon from 'vue-material-design-icons/AccountOutline.vue'
import InformationOutlineIcon from 'vue-material-design-icons/InformationOutline.vue'
import KeyOutlineIcon from 'vue-material-design-icons/KeyOutline.vue'

import SlackIcon from './icons/SlackIcon.vue'

import axios from '@nextcloud/axios'
import { showError, showSuccess } from '@nextcloud/dialogs'
import { loadState } from '@nextcloud/initial-state'
import { confirmPassword } from '@nextcloud/password-confirmation'
import { generateUrl } from '@nextcloud/router'
import NcCheckboxRadioSwitch from '@nextcloud/vue/dist/Components/NcCheckboxRadioSwitch.js'

import { delay } from '../utils.js'

export default {
	name: 'AdminSettings',

	components: {
		SlackIcon,
		NcCheckboxRadioSwitch,
		AccountOutlineIcon,
		InformationOutlineIcon,
		KeyOutlineIcon,
	},

	props: [],

	data() {
		return {
			state: loadState('integration_slack', 'admin-config'),
			// to prevent some browsers to fill fields with remembered passwords
			readonly: true,
			redirect_uri: window.location.protocol + '//' + window.location.host + generateUrl('/apps/integration_slack/oauth-redirect'),
		}
	},

	watch: {
	},

	mounted() {
	},

	methods: {
		onUsePopupChanged(newValue) {
			this.saveOptions({ use_popup: newValue ? '1' : '0' })
		},
		onInput() {
			delay(() => {
				const values = {
					client_id: (this.state.client_id ?? '').trim(),
				}
				if (this.state.client_secret !== 'dummyClientSecret') {
					values.client_secret = (this.state.client_secret ?? '').trim()
				}
				this.saveOptions(values, true)
			}, 2000)()
		},
		async saveOptions(values, sensitive = false) {
			if (sensitive) {
				await confirmPassword()
			}
			const req = {
				values,
			}
			const url = sensitive
				? generateUrl('/apps/integration_slack/sensitive-admin-config')
				: generateUrl('/apps/integration_slack/admin-config')
			axios.put(url, req).then(() => {
				showSuccess(t('integration_slack', 'Slack admin options saved'))
			}).catch((error) => {
				showError(t('integration_slack', 'Failed to save Slack admin options'))
				console.error(error)
			})
		},
	},
}
</script>

<style scoped lang="scss">
#slack_prefs {
	#slack-content {
		margin-left: 40px;
	}

	h2,
	.line,
	.settings-hint {
		display: flex;
		align-items: center;
		color: var(--text-color);
		justify-content: flex-start;
		.icon {
			margin-right: 4px;
		}
	}

	h2 .icon {
		margin-right: 8px;
	}

	.line {
		> label {
			width: 300px;
			display: flex;
			align-items: center;
		}
		> input {
			width: 300px;
		}

		&.slack-checkbox {
			margin-top: 16px;
		}
	}

	a {
		text-decoration: underline;
	}
}
</style>
