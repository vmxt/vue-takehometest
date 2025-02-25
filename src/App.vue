<template>
  <div class="p-6 w-3xl text-white rounded-lg font-sans">
    <h1 class="text-4xl font-bold text-center mb-6">VUE CRUD TAKE HOME TEST</h1>
    
    <div class="flex gap-3 mb-6">
      <input v-model.trim="newTopic" placeholder="Enter topic name"
        class="w-full p-3 border rounded-lg bg-dark text-white placeholder-gray-400 focus:ring-2 focus:ring-dark-500 focus:outline-none" />
      <button @click="addTopic" class="bg-dark-600 text-white px-5 py-3 rounded-lg hover:bg-dark-700 transition">Add</button>
    </div>
    
    <ul class="space-y-4">
      <li v-for="topic in paginatedTopics" :key="topic.guid" class="p-5 bg-dark-800 rounded-lg shadow-md">
        <div class="flex justify-between items-center">
          <span v-if="!topic.editing" @click="selectTopic(topic)"
            class="cursor-pointer font-semibold text-light hover:underline w-full overflow-hidden truncate">
            ID: {{ topic.guid }} - <span class="truncate w-60">{{ topic.name }}</span>
          </span>
          <input v-else v-model="topic.name" class="w-150 mr-2 p-1 bg-gray-700 text-white border rounded" />
          
          <div class="flex gap-2">
            <button v-if="!topic.editing" @click="editTopic(topic)" class="text-yellow-400 hover:text-yellow-500 transition">Edit</button>
            <button v-else @click="saveTopic(topic)" class="text-green-400 hover:text-green-500 transition">Save</button>
            <button @click="deleteTopic(topic.guid)" class="text-red-400 hover:text-red-500 transition">Delete</button>
          </div>
        </div>
        
        <div v-if="selectedTopic?.guid === topic.guid" class="mt-4 border-t border-gray-700 pt-3">
          <h2 class="text-lg font-semibold">Comments:</h2>
          <ul v-if="topic.comments.length" class="mt-2 space-y-2">
            <li v-for="comment in topic.comments" :key="comment.date" class="p-3 bg-dark-700 rounded-md">
              <strong class="text-yellow-400">{{ getPersonName(comment.by) }}:</strong> {{ comment.comment }}
            </li>
          </ul>
          <p v-else class="text-gray-400">No comments</p>
        </div>
      </li>
    </ul>
    
    <div class="mt-6 flex justify-center items-center gap-4">
      <button @click="prevPage" :disabled="currentPage === 1"
        class="px-4 py-2 border rounded-lg bg-gray-700 hover:bg-gray-600 disabled:opacity-50">Prev</button>
      <span class="text-lg font-semibold">Page {{ currentPage }}</span>
      <button @click="nextPage" :disabled="currentPage * itemsPerPage >= topics.length"
        class="px-4 py-2 border rounded-lg bg-gray-700 hover:bg-gray-600 disabled:opacity-50">Next</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const topics = ref([]);
const persons = ref([]);
const newTopic = ref("");
const currentPage = ref(1);
const itemsPerPage = 10;
const selectedTopic = ref(null);

const fetchData = async (url, targetRef) => {
  try {
    const response = await fetch(url);
    const data = await response.json();
    targetRef.value = data[targetRef === topics ? 'topics' : 'persons'].map(item => ({ ...item, editing: false }));
  } catch (error) {
    console.error(`Error fetching ${targetRef === topics ? 'topics' : 'persons'}`, error);
  }
};

const getPersonName = (guid) => {
  const person = persons.value.find(p => p.guid === guid);
  return person ? `${person.first} ${person.last}` : "Unknown";
};

const generateShortGuid = () => Math.random().toString(36).substring(2, 10).toUpperCase();

const addTopic = () => {
  if (newTopic.value.trim()) {
    topics.value.unshift({
      guid: generateShortGuid(),
      name: newTopic.value,
      by: "user",
      time: new Date().toISOString(),
      comments: [],
      editing: false,
    });
    newTopic.value = "";
  }
};

const deleteTopic = (guid) => {
  topics.value = topics.value.filter(topic => topic.guid !== guid);
  if (selectedTopic.value?.guid === guid) selectedTopic.value = null;
};

const selectTopic = (topic) => {
  selectedTopic.value = selectedTopic.value?.guid === topic.guid ? null : topic;
};

const editTopic = (topic) => topic.editing = true;
const saveTopic = (topic) => topic.editing = false;

const paginatedTopics = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  return topics.value.slice(start, start + itemsPerPage);
});

const nextPage = () => {
  if (currentPage.value < Math.ceil(topics.value.length / itemsPerPage)) currentPage.value++;
};

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--;
};

onMounted(() => {
  fetchData("https://atillc.blob.core.windows.net/data-collector/icode/test-data/topics.json", topics);
  fetchData("https://atillc.blob.core.windows.net/data-collector/icode/test-data/topics.json", persons);
});
</script>
