<!--
  - @copyright 2019 Christoph Wurst <christoph@winzerhof-wurst.at>
  -
  - @author 2019 Christoph Wurst <christoph@winzerhof-wurst.at>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -->

<template>
	<Modal
		size="full"
		:title="t('mail', 'Account settings')"
		@close="$emit('close')">
		<h2>{{ t('mail', 'Account settings') }}</h2>
		<p>
			<strong>{{ displayName }}</strong> &lt;{{ email }}&gt;
			<a
				v-if="!account.provisioned"
				class="button icon-rename"
				href="#account-form"
				:title="t('mail', 'Change name')" />
		</p>
		<AliasSettings v-if="!account.provisioned" :account="account" />
		<SignatureSettings :account="account" />
		<EditorSettings :account="account" />
		<div v-if="!account.provisioned" class="section">
			<h2>{{ t('mail', 'Mail server') }}</h2>
			<div id="mail-settings">
				<AccountForm
					:key="account.accountId"
					:display-name="displayName"
					:email="email"
					:save="onSave"
					:account="account" />
			</div>
		</div>
	</Modal>
</template>

<script>
import AccountForm from '../components/AccountForm'
import EditorSettings from '../components/EditorSettings'
import Logger from '../logger'
import SignatureSettings from '../components/SignatureSettings'
import AliasSettings from '../components/AliasSettings'
import Modal from '@nextcloud/vue/dist/Components/Modal'

export default {
	name: 'AccountSettings',
	components: {
		AccountForm,
		AliasSettings,
		EditorSettings,
		SignatureSettings,
		Modal,
	},
	props: {
		account: {
			type: Object,
			required: true,
		},
	},
	computed: {
		menu() {
			return this.buildMenu()
		},
		displayName() {
			return this.$store.getters.getAccount(this.$route.params.accountId).name
		},
		email() {
			return this.$store.getters.getAccount(this.$route.params.accountId).emailAddress
		},
	},
	methods: {
		onSave(data) {
			Logger.log('saving data', { data })
			return this.$store
				.dispatch('updateAccount', {
					...data,
					accountId: this.$route.params.accountId,
				})
				.then((account) => account)
				.catch((error) => {
					Logger.error('account update failed:', { error })

					throw error
				})
		},
	},
}
</script>

<style lang="scss" scoped>
.button.icon-rename {
	background-color: transparent;
	border: none;
	opacity: 0.3;

	&:hover,
	&:focus {
		opacity: 1;
	}
}
::v-deep .modal-container {
	display: block;
	overflow: scroll;
	transition: transform 300ms ease;
	border-radius: var(--border-radius-large);
	box-shadow: 0 0 40px rgba(0,0,0,0.2);
	padding: 30px 70px 20px;
}
</style>
