<template>
	<v-col cols="12" md="6">
		<v-card>
			<v-card-title>
				{{ $t('systemsettings.misc.title') }}
				<span v-if="isMiscSettingsEdited">*</span>
				<v-spacer></v-spacer>
			</v-card-title>

			<v-card-text>
				<v-row v-if="!!miscSettings">
					<v-col cols="12">
						<v-select
							@change="watchDataChange()"
							v-model="miscSettings.uiPerformance"
							:items="uiPerformanceItems"
							item-value="value"
							item-text="text"
							:label="$t('systemsettings.misc.uiPerformance')"
							dense
						></v-select>
					</v-col>
					<v-col cols="12">
                        <v-select
                            @change="watchDataChange()"
                            v-model="miscSettings.dataPointRuntimeValueSynchronized"
                            :items="dataPointRuntimeValueSynchronized"
                            item-value="value"
                            item-text="text"
                            :label="$t('systemsettings.misc.dataPointRuntimeValueSynchronized')"
                            dense
                        ></v-select>
                    </v-col>
                    <v-col cols="12">
                        <v-switch
                            v-model="miscSettings.enableFullScreen"
                            :label="$t('systemsettings.view.forceFullScreen')"
                            @change="watchDataChange()"
                        ></v-switch>
                    </v-col>
                    <v-col cols="12">
                        <v-switch
                            v-model="miscSettings.hideShortcutDisableFullScreen"
                            :label="$t('systemsettings.view.hideShortcutDisableFullScreen')"
                            @change="watchDataChange()"
                        ></v-switch>
                    </v-col>
                    <v-col cols="12">
                        <v-switch
                            v-model="miscSettings.eventPendingCacheEnabled"
                            :label="$t('systemsettings.event.pendingCacheEnabled')"
                            @change="watchDataChange()"
                        ></v-switch>
                    </v-col>
                    <v-col cols="12">
                        <v-text-field
                            v-model="miscSettings.eventPendingLimit"
                            :label="$t('systemsettings.event.pendingLimit')"
                            @change="watchDataChange()"
                            type="number"
                            dense
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                        <v-switch
                            v-model="miscSettings.workItemsReportingEnabled"
                            :label="$t('systemsettings.workitems.reporting.enabled')"
                            @change="watchDataChange()"
                        ></v-switch>
                    </v-col>
                    <v-col cols="12">
                        <v-switch
                            v-model="miscSettings.workItemsReportingItemsPerSecondEnabled"
                            :label="$t('systemsettings.workitems.reporting.itemspersecond.enabled')"
                            @change="watchDataChange()"
                        ></v-switch>
                    </v-col>
                    <v-col cols="12">
                        <v-text-field
                            v-model="miscSettings.workItemsReportingItemsPerSecondLimit"
                            :label="$t('systemsettings.workitems.reporting.itemspersecond.limit')"
                            @change="watchDataChange()"
                            type="number"
                            dense
                        ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                        <v-text-field
                            v-model="miscSettings.threadsNameAdditionalLength"
                            :label="$t('systemsettings.threads.name.additional.length')"
                            @change="watchDataChange()"
                            type="number"
                            dense
                        ></v-text-field>
                    </v-col>
                </v-row>
			</v-card-text>
		</v-card>
	</v-col>
</template>
<script>
export default {
	name: 'MiscSettingsComponent',

	data() {
		return {
			miscSettings: undefined,
			miscSettingsStore: undefined,
			isMiscSettingsEdited: false,
			uiPerformanceItems: [
				{ value: 1000, text: this.$t('systemsettings.misc.performance.veryHigh') },
				{ value: 2000, text: this.$t('systemsettings.misc.performance.high') },
				{ value: 5000, text: this.$t('systemsettings.misc.performance.medium') },
				{ value: 10000, text: this.$t('systemsettings.misc.performance.low') },
			],
			dataPointRuntimeValueSynchronized: [
                { value: 'NONE', text: this.$t('systemsettings.misc.dataPointRuntimeValueSynchronized.none')  },
                { value: 'PARTIAL', text: this.$t('systemsettings.misc.dataPointRuntimeValueSynchronized.partial') },
                { value: 'ALL', text: this.$t('systemsettings.misc.dataPointRuntimeValueSynchronized.all') },
            ]
		};
	},

	mounted() {
		this.fetchData();
	},

	methods: {
		async fetchData() {
			this.miscSettings = await this.$store.dispatch('getMiscSettings');
			this.miscSettingsStore = this.copyDataFromStore();
		},

		saveData() {
			console.log('Saved!');
			this.$store.commit('setMiscSettings', this.miscSettings);
			this.miscSettingsStore = this.copyDataFromStore();
			this.$store
				.dispatch('saveMiscSettings')
				.then((resp) => {
					if (resp) {
						this.restoreData();
						this.$store.dispatch('showSuccessNotification', this.$t('systemsettings.notification.save.misc'));
					}
				})
				.catch(() => {
					this.$store.dispatch('showErrorNotification', this.$t('systemsettings.notification.fail'));
				});
		},

		restoreData() {
			this.fetchData();
			this.isMiscSettingsEdited = false;
		},

		copyDataFromStore() {
			return JSON.parse(JSON.stringify(this.$store.state.systemSettings.miscSettings));
		},

		async watchDataChange() {
			this.isMiscSettingsEdited = await this.isDataChanged();
			this.emitData(this.isMiscSettingsEdited);
		},

		async isDataChanged() {
			return !(await this.$store.dispatch('configurationEqual', {
				object1: this.miscSettings,
				object2: this.miscSettingsStore,
			}));
		},

		emitData(changed) {
			this.$emit('changed', {
				component: 'miscSettingsComponent',
				title: 'systemsettings.misc.title',
				changed: changed,
				data: this.sumarizeDataChanges(),
			});
		},

		sumarizeDataChanges() {
			let data = [];
			for (let key in this.miscSettings) {
				if (this.miscSettings[key] !== this.miscSettingsStore[key]) {
					data.push({
						label: `systemsettings.misc.${key}`,
						originalData: this.convertTimePeriod(this.miscSettingsStore[key], key),
						changedData: this.convertTimePeriod(this.miscSettings[key], key),
					});
				}
			}
			return data;
		},

		purgeData() {
			this.$confirm({
				title: this.$t('systemsettings.alert.purgedata.title'),
				content: this.$t('systemsettings.alert.purgedata'),
			}).then(() => {
				this.$store.dispatch('purgeData').then((resp) => {
					if (resp === true) {
						this.$store.dispatch('showSuccessNotification', this.$t('systemsettings.notification.purgedata'));
					}
				});
			});
		},

		convertTimePeriod: function (value, key) {
			if (key.includes('Type')) {
				switch (Number(value)) {
					case 2:
						return this.$t('timeperiod.minutes');
					case 3:
						return this.$t('timeperiod.hours');
					case 4:
						return this.$t('timeperiod.days');
					case 5:
						return this.$t('timeperiod.weeks');
					case 6:
						return this.$t('timeperiod.months');
					case 7:
						return this.$t('timeperiod.years');
				}
			}
			return value;
		},
	},
};
</script>
<style></style>
