<template>
    <div v-if="allActions.length">
        <div class="hidden md:flex justify-end items-center gap-2">
            <ActionDropdown
                    v-if="invisibleActions.length"
                    class="mr-2"
                    :resource-name="resourceName"
                    :viaRelationship="viaRelationship"
                    :viaResource="viaResource"
                    :viaResourceId="viaResourceId"
                    :actions="invisibleActions"
            />
            <ActionButtons
                    v-if="visibleActions.length"
                    :resource-name="resourceName"
                    :viaRelationship="viaRelationship"
                    :viaResource="viaResource"
                    :viaResourceId="viaResourceId"
                    :actions="visibleActions"
                    @actionExecuted="$emit('actionExecuted')"/>
        </div>
        <div class="flex md:hidden">
            <ActionDropdown
                    class="mr-2"
                    :resource-name="resourceName"
                    :viaRelationship="viaRelationship"
                    :viaResource="viaResource"
                    :viaResourceId="viaResourceId"
                    :actions="allActions"
            />
        </div>
    </div>
</template>
<script setup>

import find from 'lodash/find';
import filter from 'lodash/filter';
import {onMounted, ref} from "vue";


const props = defineProps({
    resourceName: {},
    viaResource: {},
    viaResourceId: {},
    viaRelationship: {},
    relationshipType: {},
    selectedResources: {type: [Array, String], default: () => []},
    endpoint: {type: String, default: null},
})

const visibleActions = ref([]);
const invisibleActions = ref([]);
const allActions = ref([]);

onMounted(() => {
    getActions();
});

const getActions = () => {
    Nova.request()
        .get(`/nova-api/${props.resourceName}/actions`, {
            params: {
                viaResource: props.viaResource,
                viaResourceId: props.viaResourceId,
                viaRelationship: props.viaRelationship,
                relationshipType: props.relationshipType
            }
        })
        .then(response => {
            allActions.value = filter(
                response.data.actions,
                (action) => action.detachedAction
            );

            let visibleActionsLimit = find(Nova.config('resources'), resource => {
                return resource.uriKey === props.resourceName
            }).visibleActionsLimit ?? 3

            visibleActions.value = (visibleActionsLimit === 0) ? [] : allActions.value.slice(0, visibleActionsLimit).reverse()
            invisibleActions.value = allActions.value.slice(visibleActionsLimit);

        })
};

</script>
