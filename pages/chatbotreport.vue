<template>
  <div class="full">
    <Header class="Header" />

    <!-- Table selection buttons -->
    <div class="primary-table">
      <div class="button-group">
        <button type="button" class="primary-btn" :class="{ 'selected': selectedTable === 'patientBooking' }"
          @click="toggleTable('patientBooking')">Patient Booking</button>
        <button type="button" class="secondary-btn" :class="{ 'selected': selectedTable === 'queriedPatient' }"
          @click="toggleTable('queriedPatient')">Queried Patient</button>
        <button type="button" class="third-btn" :class="{ 'selected': selectedTable === 'rejectedPatient' }"
          @click="toggleTable('rejectedPatient')">Rejected Patient</button>
        <button type="button" class="fourth-btn" :class="{ 'selected': selectedTable === 'sentnumbers' }"
          @click="toggleTable('sentnumbers')">Sent Message</button>
        <button type="button" class="fifth-btn" :class="{ 'selected': selectedTable === 'failednumbers' }"
          @click="toggleTable('failednumbers')">Failed Numbers</button>
      </div>
    </div>

    <!-- Date selection and search -->
    <div class="top">
      <div class="expected-date">
        <label for="chooseDate">Choose the Date</label>
        <input type="date" id="chooseDate" v-model="searchDate" @input="updateDate" class="custom-input" />
        <span class="search-icon" @click="searchReport">&#128269;</span>
      </div>
      <div class="search-input">

      </div>
    </div>

    <!-- Patient Booking Table -->
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'patientBooking' }">
      <table v-if="selectedTable === 'patientBooking'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>Patient Choice</th>
            <th>Date and Time</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="3" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="patientchoice">{{ item.doctor_choice }}</td>
            <td data-title="timestamp">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Queried Patient Table -->
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'queriedPatient' }">
      <table v-if="selectedTable === 'queriedPatient'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>Date and Time</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="2" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="timestamp">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Rejected Patient Table -->
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'rejectedPatient' }">
      <table v-if="selectedTable === 'rejectedPatient'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>Date and Time</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="2" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="timestamp">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Sent message Table -->
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'sentnumbers' }">
      <table v-if="selectedTable === 'sentnumbers'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>Date and Time</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="2" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="date">{{ item.date }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="table-container" :class="{ 'selected-table': selectedTable === 'failednumbers' }">
      <table v-if="selectedTable === 'failednumbers'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>Date and Time</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="2" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="date">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <nav aria-label="Page navigation" class="pagination-container">
      <ul class="pagination">
        <li class="page-item">
          <a class="page-link" @click="changePage(pagination.page - 1)" href="#" :disabled="pagination.page === 1">
            &lt;
          </a>
        </li>
        <li class="page-item" :class="{ active: pagination.page === page }" v-for="page in visiblePages" :key="page">
          <a class="page-link" @click="changePage(page)" href="#">{{ page }}</a>
        </li>
        <li class="page-item">
          <a class="page-link" @click="changePage(pagination.page + 1)" href="#"
            :disabled="pagination.page === totalPages">
            &gt;
          </a>
        </li>
      </ul>
    </nav>
    <Footer />
  </div>
</template>

<script>
import { ref, watch, onMounted, computed } from 'vue';
import Header from '~/components/Header.vue';
import { getReportsByDate } from '@/api/api.js';

export default {
  components: {
    Header,
  },
  setup() {
    const searchDate = ref(new Date().toISOString().split('T')[0]);
    const dataToDisplay = ref([]);
    const pagination = ref({
      page: 1,
      itemsPerPage: 9,
    });
    const originalReportData = ref([]);
    const selectedTable = ref('patientBooking'); // Set a default table name

    // Declare the arrays
    const queriedPatients = ref([]);
    const noAppointments = ref([]);
    const currentPatients = ref([]);
    const sentnumbers = ref([]);
    const failednumbers = ref([]);

    const searchReport = async () => {
      if (!searchDate.value) {
        console.error('Please choose a date.');
        return;
      }

      try {
        console.log('Making API request with date:', searchDate.value);
        const response = await getReportsByDate(searchDate.value);

        // Store each category in separate arrays with a deep copy
        originalReportData.value = response;
        queriedPatients.value = JSON.parse(JSON.stringify(originalReportData.value.queriedPatients || []));
        console.log(queriedPatients.value);
        noAppointments.value = JSON.parse(JSON.stringify(originalReportData.value.noAppointments || []));
        console.log(noAppointments.value);
        currentPatients.value = JSON.parse(JSON.stringify(originalReportData.value.currentPatients || []));
        console.log(currentPatients.value);
        sentnumbers.value = JSON.parse(JSON.stringify(originalReportData.value.sentnumbers || []));
        console.log(sentnumbers.value);
        failednumbers.value = JSON.parse(JSON.stringify(originalReportData.value.failedPatients || []));
        console.log(failednumbers.value);

        pagination.value.page = 1;
      } catch (error) {
        console.error('Error fetching data:', error.message);
      }
    };

    onMounted(async () => {
      // Make the initial API call when the component is mounted
      await searchReport();
    });

    // Watch for changes in originalReportData and update dataToDisplay accordingly
    watch([originalReportData, selectedTable], ([originalData, table]) => {
      switch (table) {
        case 'patientBooking':
          dataToDisplay.value = [...currentPatients.value];
          break;
        case 'queriedPatient':
          dataToDisplay.value = [...queriedPatients.value];
          break;
        case 'rejectedPatient':
          dataToDisplay.value = [...noAppointments.value];
          break;
        case 'sentnumbers':
          dataToDisplay.value = [...sentnumbers.value];
	  dataToDisplay.value.sort((a, b) => new Date(b.date) - new Date(a.date));
          break;
        case 'failednumbers':
          dataToDisplay.value = [...failednumbers.value];
          break;
      }
      dataToDisplay.value.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
    });

    const updateDate = (event) => {
      searchDate.value = event.target.value;
    };

    const toggleTable = (tableName) => {
      selectedTable.value = tableName;
      pagination.value.page = 1;
    };

    const changePage = (page) => {
      if (page >= 1 && page <= totalPages.value) {
        pagination.value.page = page;
      }
    };

    const paginatedItems = computed(() => {
      const dataToPaginate = Array.isArray(dataToDisplay.value) ? dataToDisplay.value : [];
      const start = (pagination.value.page - 1) * pagination.value.itemsPerPage;
      const end = start + pagination.value.itemsPerPage;
      return dataToPaginate.slice(start, end);
    });

    const totalPages = computed(() => {
      const dataToPaginate = Array.isArray(dataToDisplay.value) ? dataToDisplay.value : [];
      return Math.ceil(dataToPaginate.length / pagination.value.itemsPerPage);
    });

    const visiblePages = computed(() => {
      const currentPage = pagination.value.page;
      const lastPage = totalPages.value;
      const limit = 5; // Adjust this value to change the number of visible pages

      let start = Math.max(1, currentPage - Math.floor(limit / 2));
      let end = Math.min(lastPage, start + limit - 1);

      if (lastPage - end < Math.floor(limit / 2)) {
        start = Math.max(1, lastPage - limit + 1);
      }

      return Array.from({ length: end - start + 1 }, (_, i) => i + start);
    });

    return {
      searchDate,
      dataToDisplay,
      pagination,
      originalReportData,
      selectedTable,
      searchReport,
      updateDate,
      toggleTable,
      changePage,
      paginatedItems,
      totalPages,
      visiblePages,
    };
  },
};
</script>

<style scoped>
@import '@/styles/chatbot.css';
</style>
