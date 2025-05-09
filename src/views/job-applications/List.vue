<template>
    <div class="p-4">
        <div class="flex flex-col sm:flex-row gap-4 mb-4">
            <input type="text" v-model="searchQuery" placeholder="Search by text..."
                class="border p-2 w-full sm:w-1/2" />

            <input type="date" v-model="selectedDate" class="border p-2 w-full sm:w-1/2" />

            <select v-model="selectedProfileName" class="border p-2 w-full sm:w-1/3">
                <option value="">All Profiles</option>
                <option v-for="(profile, index) in uniqueProfileNames" :key="index" :value="profile">
                    {{ profile || 'Unnamed Profile' }}
                </option>
            </select>

            <!-- Employee Name Dropdown -->
            <select v-model="selectedEmployeeName" class="border p-2 w-full sm:w-1/4">
                <option value="">All Employees</option>
                <option v-for="(employee, index) in uniqueEmployeeNames" :key="'employee-' + index" :value="employee">
                    {{ employee || 'Unnamed Employee' }}
                </option>
            </select>

            <!-- Platform Dropdown -->
            <select v-model="selectedPlatform" class="border p-2 w-full sm:w-1/5">
                <option value="">All Platforms</option>
                <option v-for="(platform, index) in uniquePlatforms" :key="'platform-' + index" :value="platform">
                    {{ platform || 'Unnamed Platform' }}
                </option>
            </select>
        </div>
        <div>
            <h4>Count : {{ filteredData.length }}</h4>
        </div>
        <div class="overflow-x-auto">
            <table class="min-w-full bg-white border border-gray-300" >
                <thead class="bg-gray-100">
                    <tr>
                        <th class="py-2 px-4 border-b">Date</th>
                        <th class="py-2 px-4 border-b">Job Title</th>
                        <th class="py-2 px-4 border-b">Company</th>
                        <th class="py-2 px-4 border-b">Platform</th>
                        <th class="py-2 px-4 border-b">Profile Name</th>
                        <th class="py-2 px-4 border-b">Employee Name</th>
                        <th class="py-2 px-4 border-b">Status</th>
                        <th class="py-2 px-4 border-b">Link</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in filteredData" :key="index" class="hover:bg-gray-300">
                        <td class="py-2 px-4 border-b">{{ formatDate(item.Date) }}</td>
                        <td class="py-2 px-4 border-b" :title="item['Job Title']">{{ item['Job Title'].substr(0, 50) }}</td>
                        <td class="py-2 px-4 border-b" :title="item.Company">{{ item.Company.substr(0, 50) }}</td>
                        <td class="py-2 px-4 border-b">{{ item.Platform }}</td>
                        <td class="py-2 px-4 border-b">{{ item['Profile Name'] }}</td>
                        <td class="py-2 px-4 border-b">{{ item['Employee Name'] }}</td>
                        <td class="py-2 px-4 border-b">{{ item.Status }}</td>
                        <td class="py-2 px-4 border-b">
                            <a :href="item.JobURL" target="_blank" class="text-blue-500 underline">View</a>
                        </td>
                    </tr>

                    <tr v-if="filteredData.length === 0">
                        <td colspan="8" class="text-center py-4">No results found.</td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div>
            <h4>Count : {{ filteredData.length }}</h4>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const data = ref([])
const searchQuery = ref('')
const selectedDate = ref('')
const selectedProfileName = ref('')
const selectedEmployeeName = ref('')
const selectedPlatform = ref('')



// Fetch data from your Google Apps Script API
const fetchData = async () => {
    fetch("https://script.google.com/macros/s/AKfycbzdHmKt224W4Q-A2yxcbrGTBF3kTOIFoILWDpnRnPz8B0kgTO-5skJN8QDy88o9dmG8Cw/exec")
        .then(response => response.json())
        .then(response => {
            data.value = response
            console.log("Fetched Records:", data);
        })
        .catch(console.error);

}

// Filtered Data
const filteredData = computed(() => {
    return data.value.filter(item => {
        const matchesSearch = searchQuery.value
            ? (item['Job Title'] && item['Job Title'].toLowerCase().includes(searchQuery.value.toLowerCase())) ||
            (item.Company && item.Company.toLowerCase().includes(searchQuery.value.toLowerCase())) ||
            (item.Platform && item.Platform.toLowerCase().includes(searchQuery.value.toLowerCase())) ||
            (item['Profile Name'] && item['Profile Name'].toLowerCase().includes(searchQuery.value.toLowerCase())) ||
            (item['Employee Name'] && item['Employee Name'].toLowerCase().includes(searchQuery.value.toLowerCase())) ||
            (item.Status && item.Status.toLowerCase().includes(searchQuery.value.toLowerCase()))
            : true

        const matchesDate = selectedDate.value
            ? formatInputDate(item.Date) === selectedDate.value
            : true

        const matchesProfileName = selectedProfileName.value
            ? item['Profile Name'] === selectedProfileName.value
            : true

        const matchesEmployeeName = selectedEmployeeName.value
            ? item['Employee Name'] === selectedEmployeeName.value
            : true

        const matchesPlatform = selectedPlatform.value
            ? item.Platform === selectedPlatform.value
            : true

        return matchesSearch && matchesDate && matchesProfileName && matchesEmployeeName && matchesPlatform
    })
})

// Unique Profile Names for Dropdown
const uniqueProfileNames = computed(() => {
    const profiles = data.value.map(item => item['Profile Name'] || '')
    const unique = [...new Set(profiles)]
    return unique.filter(profile => profile !== '')
})

// Unique Employee Names for Dropdown
const uniqueEmployeeNames = computed(() => {
    const employees = data.value.map(item => item['Employee Name'] || '')
    const unique = [...new Set(employees)]
    return unique.filter(employee => employee !== '')
})

// Unique Platforms for Dropdown
const uniquePlatforms = computed(() => {
    const platforms = data.value.map(item => item.Platform || '')
    const unique = [...new Set(platforms)]
    return unique.filter(platform => platform !== '')
})

// Format Date
const formatDate = (isoDate) => {
    const options = {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        hour12: true, // set to false for 24-hour format
    };
    return new Date(isoDate).toLocaleDateString(undefined, options);
};

const formatInputDate = (isoDate) => {
    const d = new Date(isoDate)
    // Convert to local date (Pakistan Time)
    const year = d.getFullYear()
    const month = String(d.getMonth() + 1).padStart(2, '0')
    const day = String(d.getDate()).padStart(2, '0')
    return `${year}-${month}-${day}` // format yyyy-MM-dd
}

onMounted(() => {
    fetchData()
})
</script>

<style scoped>
@media (max-width: 640px) {
    table {
        font-size: 14px;
    }
}
</style>