<template>
  <!-- Expansion item for showing/hiding content -->
  <q-expansion-item
    v-model="expanded"
    expand-icon-class="text-white"
    bordered
    class="rounded-borders shadow-1 col"
    :header-class="`${!isRoot ? 'bg-secondary' : 'bg-primary'} text-white`"
    dense
  >
    <div class="q-pa-md">
      <!-- Display message if there are no objects -->
      <div
        v-if="localValue.length === 0"
        class="no-objects-message"
      >
        No objects present yet, click on the button to add one!
      </div>
      <!-- Iterate through localValue to show individual objects -->
      <div
        v-for="(item, index) in localValue"
        :key="index"
      >
        <!-- Card to display the attributes of the object -->
        <q-card class="object-card">
          <!-- Render the attributes list for the current object -->
          <attributes-list
            :attributes="getSubAttributes(item)"
            :plugin="plugin"
            :full-name="`${fullName}.${index}`"
            :current-error="currentError"
            @update:attributes="(event) => updateObject(index, event)"
          />
          <!-- Button to delete the current object -->
          <q-btn
            flat
            round
            color="negative"
            icon="fa-solid fa-trash-alt"
            class="object-delete-btn"
            @click="removeObject(index)"
          >
            <!-- Tooltip to describe the delete button -->
            <q-tooltip
              anchor="top left"
              self="bottom right"
            >
              Delete Object
            </q-tooltip>
          </q-btn>
        </q-card>
      </div>
      <!-- Button to add a new object -->
      <q-btn
        flat
        round
        color="primary"
        icon="fa-solid fa-plus"
        class="q-mt-md"
        @click="addObject"
      >
        Add an Object
      </q-btn>
    </div>
  </q-expansion-item>
</template>

<script setup>
import { computed, ref, watch } from 'vue';
import { ComponentAttribute } from 'leto-modelizer-plugin-core';
import AttributesList from 'src/components/inputs/AttributesList.vue';

// Props and Emits are imported using the setup() function and do not require explicit comments.
const props = defineProps({
  attribute: {
    type: Object,
    required: true,
  },
  plugin: {
    type: Object,
    required: true,
  },
  isRoot: {
    type: Boolean,
    default: false,
  },
  fullName: {
    type: String,
    required: true,
  },
  currentError: {
    type: String,
    default: null,
  },
});

const expanded = ref(false);
const hasError = computed(() => props.currentError !== null && props.currentError.startsWith(`${props.fullName}.`));

// An array to store local objects (initialized as an empty array).
const localValue = ref([]);

// Watch for changes in hasError to update expanded state
watch(() => hasError.value, () => {
  expanded.value = hasError.value || expanded.value;
});

/**
 * Function to add a new object to the localValue array.
 */
function addObject() {
  // Find the definition of the object type from the attribute definition
  const objectDefinition = props.attribute.definition.definedAttributes.find(({ type }) => type === 'Object');

  // Create a new object with the definition
  const newObj = new ComponentAttribute({
    name: objectDefinition.name,
    type: objectDefinition.type,
    definition: objectDefinition,
  });

  // Add the new object to the localValue array
  localValue.value.push(newObj);

  // Set the expanded state for the new object to true
  expanded.value = true;
}

/**
 * Function to get all sub-attributes of the provided attribute.
 * @param {ComponentAttribute} attribute - Attribute.
 * @return {ComponentAttribute[]} List with all sub-attributes.
 */
function getSubAttributes(attribute) {
  const attributes = [];
  const attributeValue = attribute.value || [];

  attribute.definition?.definedAttributes?.forEach((definition) => {
    const attr = attributeValue.find(({ name }) => name === definition.name);

    if (attr) {
      attributes.push(attr);
    } else {
      attributes.push(new ComponentAttribute({
        name: definition.name,
        type: definition.type,
        definition,
      }));
    }
  });

  return [
    ...attributes,
    ...attributeValue.filter(({ definition }) => !definition),
  ];
}

/**
 * Function to update an object at a specified index with new attributes.
 * @param {number} index - Index of the object to update.
 * @param {ComponentAttribute} updatedAttributes - Updated attributes for the object.
 */
function updateObject(index, updatedAttributes) {
  localValue.value[index] = updatedAttributes;
}

/**
 * Function to remove an object at a specified index from the localValue array.
 * @param {number} index - Index of the object to remove.
 */
function removeObject(index) {
  localValue.value.splice(index, 1);
}
</script>

<!-- Scoped styles for the component. -->
<style scoped>
.object-card {
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 8px;
}

.object-delete-btn {
  position: absolute;
  bottom: 8px;
  right: 8px;
}

.no-objects-message {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100px;
  background-color: #f7f7f7;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  color: #888;
  padding: 16px;
  margin-bottom: 16px;
}
</style>
