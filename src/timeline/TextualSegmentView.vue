<template>
  <div
    v-if="activeTextualSegment"
    class="flex flex-col w-full h-full border border-zinc-600 p-6 bg-zinc-800 rounded-lg relative items-center">
    <template v-if="!isEditing">
      <button
        class="absolute top-4 right-6 px-3 py-1 rounded bg-blue-700 hover:bg-blue-600 text-white text-sm font-semibold transition z-10"
        @click="startEdit"
        data-testid="inline-edit-btn"
      >Edit</button>
    </template>
    <template v-if="isEditing">
      <div class="w-full max-w-2xl flex flex-col gap-4">
        <label class="text-zinc-300 font-semibold">Title</label>
        <input v-model="editTitle" class="w-full rounded bg-zinc-900 text-zinc-100 p-2 border border-zinc-700" />
        <label class="text-zinc-300 font-semibold">Content</label>
        <textarea v-model="editContent" class="w-full rounded bg-zinc-900 text-zinc-100 p-2 border border-zinc-700 min-h-[80px]" />
        <label class="text-zinc-300 font-semibold">Sources</label>
        <div class="flex flex-col gap-2">
          <div v-for="(src, idx) in editSources" :key="idx" class="flex gap-2 items-center">
            <input v-model="editSources[idx]" class="flex-1 rounded bg-zinc-900 text-zinc-100 p-2 border border-zinc-700" />
            <button @click="removeSource(idx)" class="text-red-400 hover:text-red-300 px-2">Remove</button>
          </div>
          <button @click="addSource" class="text-blue-400 hover:text-blue-300 text-sm mt-2">+ Add Source</button>
        </div>
        <div class="flex gap-2 mt-4">
          <button @click="saveEdit" class="px-4 py-2 rounded bg-blue-700 hover:bg-blue-600 text-white font-semibold transition">Save</button>
          <button @click="cancelEdit" class="px-4 py-2 rounded bg-zinc-700 hover:bg-zinc-600 text-white font-semibold transition">Cancel</button>
        </div>
      </div>
    </template>
    <template v-else>
      <div class="text-2xl font-bold text-center mb-2">
        {{ activeTextualSegment.title }}
      </div>
      <hr class="border-zinc-700 mb-4 w-full max-w-2xl" />
      <div class="flex flex-col items-center justify-center w-full">
        <div class="text-4xl">
          {{ activeTextualSegment.content }}
        </div>
      </div>
      <div
        class="absolute bottom-4 right-6 text-2xl bg-zinc-900 bg-opacity-80 px-4 py-2 rounded shadow text-blue-200 select-none">
        {{ activeTextualSegment.getRemainingTime() }}s
      </div>
      <div class="w-full max-w-2xl mt-4">
        <div class="text-lg text-zinc-300 mb-2">Sources:</div>
        <template v-if="activeTextualSegment.sources && activeTextualSegment.sources.length">
          <ul class="list-disc list-inside space-y-1">
            <li v-for="(src, idx) in activeTextualSegment.sources" :key="idx">
              <a :href="src" target="_blank" rel="noopener" class="text-blue-400 underline hover:text-blue-200">{{ src }}</a>
            </li>
          </ul>
        </template>
        <template v-else>
          <div class="text-zinc-400 italic">No sources provided</div>
        </template>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useTimelineStore } from '@/timeline/stores/timelineStore';
import { storeToRefs } from 'pinia';

const timelineStore = useTimelineStore();
const { timeline } = storeToRefs(timelineStore);

const activeTextualSegment = computed(() => timeline.value?.getActiveTextSegment() ?? null);

const editTitle = ref('');
const editContent = ref('');
const editSources = ref<string[]>([]);
const isEditing = ref(false);

function startEdit() {
  if (!activeTextualSegment.value) return;
  editTitle.value = activeTextualSegment.value.title;
  editContent.value = activeTextualSegment.value.content;
  editSources.value = [...(activeTextualSegment.value.sources ?? [])];
  isEditing.value = true;
}

function cancelEdit() {
  isEditing.value = false;
}

function saveEdit() {
  if (!activeTextualSegment.value) return;
  activeTextualSegment.value.title = editTitle.value;
  activeTextualSegment.value.content = editContent.value;
  activeTextualSegment.value.sources = [...editSources.value];
  isEditing.value = false;
}

function addSource() {
  editSources.value.push('');
}

function removeSource(idx: number) {
  editSources.value.splice(idx, 1);
}
</script>
