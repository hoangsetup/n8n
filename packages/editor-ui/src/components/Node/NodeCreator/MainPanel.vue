<template>
	<div
		class="container"
		ref="mainPanelContainer"
	>
		<div class="main-panel">
			<trigger-helper-panel
				v-if="selectedType === TRIGGER_NODE_FILTER"
				:searchItems="searchItems"
				@nodeTypeSelected="nodeType => $emit('nodeTypeSelected', nodeType)"
			>
				<type-selector slot="header" />
			</trigger-helper-panel>
			<categorized-items
				v-else
				:searchItems="searchItems"
				:excludedSubcategories="[OTHER_TRIGGER_NODES_SUBCATEGORY]"
				:initialActiveCategories="[CORE_NODES_CATEGORY]"
				@nodeTypeSelected="nodeType => $emit('nodeTypeSelected', nodeType)"
			>
				<type-selector slot="header" />
			</categorized-items>
		</div>
	</div>
</template>

<script lang="ts">
import { PropType } from 'vue';
import { externalHooks } from '@/components/mixins/externalHooks';
import mixins from 'vue-typed-mixins';
import TriggerHelperPanel from './TriggerHelperPanel.vue';
import { ALL_NODE_FILTER, TRIGGER_NODE_FILTER, OTHER_TRIGGER_NODES_SUBCATEGORY, CORE_NODES_CATEGORY } from '@/constants';
import CategorizedItems from './CategorizedItems.vue';
import TypeSelector from './TypeSelector.vue';
import { INodeCreateElement } from '@/Interface';

export default mixins(externalHooks).extend({
	name: 'NodeCreateList',
	components: {
		TriggerHelperPanel,
		CategorizedItems,
		TypeSelector,
	},
	props: {
		searchItems: {
			type: Array as PropType<INodeCreateElement[] | null>,
		},
	},
	data() {
		return {
			CORE_NODES_CATEGORY,
			TRIGGER_NODE_FILTER,
			ALL_NODE_FILTER,
			OTHER_TRIGGER_NODES_SUBCATEGORY,
		};
	},
	computed: {
		selectedType(): string {
			return this.$store.getters['nodeCreator/selectedType'];
		},
	},
	watch: {
		selectedType(newValue, oldValue) {
			this.$externalHooks().run('nodeCreateList.selectedTypeChanged', {
				oldValue,
				newValue,
			});
			this.$telemetry.trackNodesPanel('nodeCreateList.selectedTypeChanged', {
				old_filter: oldValue,
				new_filter: newValue,
				workflow_id: this.$store.getters.workflowId,
			});
		},
	},
	mounted() {
		this.$externalHooks().run('nodeCreateList.mounted');
		// Make sure tabs are visible on mount
		this.$store.commit('nodeCreator/setShowTabs', true);
	},
	destroyed() {
		this.$store.commit('nodeCreator/setSelectedType', ALL_NODE_FILTER);
		this.$externalHooks().run('nodeCreateList.destroyed');
		this.$telemetry.trackNodesPanel('nodeCreateList.destroyed', { workflow_id: this.$store.getters.workflowId });
	},
});
</script>
<style lang="scss" scoped>
.container {
	height: 100%;
}
.main-panel {
	height: 100%;
}
</style>
