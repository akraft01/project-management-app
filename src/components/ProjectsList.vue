<template>
  <div class="max-w-6xl mx-auto py-10 px-4 lg:px-8 bg-blue-200 rounded-lg shadow-lg">
    <h1 class="text-4xl font-bold mb-10 text-center text-blue-700">Projects</h1>
    <!-- Barre de Recherche et de Tri -->
    <div class="mb-8 flex flex-col md:flex-row justify-between items-center gap-4">
      <!-- Barre de Recherche -->
      <div class="relative w-full md:w-1/2">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search for a project..."
          class="w-full p-3 pl-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-teal-500"
        />
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400"
          viewBox="0 0 20 20"
          fill="currentColor"
        >
          <path
            fill-rule="evenodd"
            d="M12.9 14.32a7.5 7.5 0 111.414-1.414l5.296 5.297a1 1 0 01-1.414 1.414l-5.296-5.297zM10 15a5 5 0 100-10 5 5 0 000 10z"
            clip-rule="evenodd"
          />
        </svg>
      </div>
      <!-- Menu de Tri -->
      <select
        v-model="sortOption"
        class="w-full md:w-1/4 p-3 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-teal-500"
      >
        <option value="name">Sort by Name</option>
        <option value="peCode">Sort by PE Code</option>
      </select>
    </div>
    <div v-if="filteredAndSortedProjects.length === 0" class="text-center text-gray-500">
      <p>No projects found.</p>
    </div>
    <div v-else class="flex flex-col space-y-8">
      <div
        v-for="project in filteredAndSortedProjects"
        :key="project.id"
        class="p-6 border rounded-lg shadow-lg bg-gradient-to-br from-purple-100 via-blue-100 to-green-100 hover:shadow-2xl transition-shadow duration-300 cursor-pointer"
        @click="openProjectDetails(project)"
      >
        <h2 class="text-2xl font-bold mb-2 text-blue-800">{{ project.name }}</h2>
        <div class="mb-3 flex flex-wrap items-center space-x-2">
          <span :class="['inline-block px-2 py-1 text-xs font-semibold rounded-full', visibilityBadge(project.visibility)]">
            {{ project.visibility }}
          </span>
          <span class="text-sm font-medium text-gray-600">| PE Code: {{ project.peCode }} | WID: {{ project.wid }}</span>
        </div>
        <p class="mb-4 text-gray-700"><strong>Description:</strong> <span v-html="project.descriptionHtml"></span></p>
      </div>
    </div>

    <!-- Modal pour la Vue Détail -->
    <div v-if="selectedProject" class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center">
      <div class="bg-blue-100 max-w-xl p-6 rounded-lg shadow-lg relative">
        <button @click="closeProjectDetails" class="absolute top-2 right-2 text-gray-500 hover:text-gray-800">
          &times;
        </button>
        <h2 class="text-3xl font-bold mb-4 text-blue-500">{{ selectedProject.name }}</h2>
        <p class="mb-4"><strong>Number of Risks:</strong> {{ selectedProject.risks.length }}</p>
        <p class="mb-4"><strong>Number of Milestones:</strong> {{ selectedProject.milestones.length }}</p>
        <p class="mb-4"><strong>Contact:</strong> {{ selectedProject.contact }}</p>
        <p class="mb-4"><strong>Business Partner Contact:</strong> {{ selectedProject.businessPartnerContact }}</p>
        <p class="mb-4"><strong>Business Partner Division FRC:</strong> {{ selectedProject.businessPartnerDivisionFrc }}</p>
        <p class="mb-4"><strong>DPMO Jira Key:</strong> {{ selectedProject.dpmoJiraKey }}</p>
        <p class="mb-4"><strong>Last Updated:</strong> {{ selectedProject.status?.lastUpdated }}</p>
        <a :href="selectedProject.webUrl" class="inline-block mt-4 text-blue-600 hover:text-teal-600 transition-colors duration-200" target="_blank">
          <svg xmlns="http://www.w3.org/2000/svg" class="inline w-5 h-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10.293 3.293a1 1 0 011.414 0l5 5a1 1 0 01-1.414 1.414L11 5.414V15a1 1 0 11-2 0V5.414L4.707 9.707a1 1 0 01-1.414-1.414l5-5z" clip-rule="evenodd"/>
          </svg>
          Visit Project
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const projects = ref([]);
const searchQuery = ref("");
const sortOption = ref("name");
const selectedProject = ref(null);

const filteredProjects = computed(() =>
  projects.value.filter((project) =>
    project.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
);

const filteredAndSortedProjects = computed(() => {
  const sorted = [...filteredProjects.value];
  switch (sortOption.value) {
    case "peCode":
      return sorted.sort((a, b) => a.peCode.localeCompare(b.peCode));
    case "name":
    default:
      return sorted.sort((a, b) => a.name.localeCompare(b.name));
  }
});

const openProjectDetails = (project) => {
  selectedProject.value = project;
};

const closeProjectDetails = () => {
  selectedProject.value = null;
};

const fetchProjects = async () => {
  try {
    const response = await fetch("/projects.json");
    projects.value = await response.json();
  } catch (error) {
    console.error("Error fetching projects:", error);
  }
};

const visibilityBadge = (visibility) => {
  switch (visibility) {
    case "public":
      return "bg-green-200 text-green-900";
    case "private":
      return "bg-red-200 text-red-900";
    case "internal":
      return "bg-yellow-200 text-yellow-900";
    default:
      return "bg-gray-200 text-gray-900";
  }
};

onMounted(fetchProjects);
</script>