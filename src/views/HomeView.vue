<template>
  <section class="text-black dark:text-white bg-white dark:bg-base-100 body-font">
    <div v-if="!groupedData" class="container px-5 py-12 mx-auto">
      <div class="flex flex-col items-center space-y-5">
        <h1 class="text-2xl font-medium title-font">
          Data Absensi Karyawan
        </h1>
      </div>
    </div>
    <div v-else class="container px-5 py-12 mx-auto">
      <div class="flex flex-col items-center space-y-5">
        <h1 class="text-2xl font-medium title-font">
          Input Data Absensi
        </h1>
        <input
          type="file"
          @change="handleFileChange"
          class="file-input text-white file-input-info dark:file-input-success "
          accept=".xlsx,.xls"
        />
      </div>
    </div>
     
  </section>

  <div class="text-black dark:text-white bg-white dark:bg-base-100 container mx-auto p-4">
    <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-4 gap-4">
      <div class="flex space-x-4">
        <button 
          class="btn btn-success text-black dark:text-white bg-white dark:bg-base-100"
          @click="saveDataToLocalstorage"
          :disabled="!excelData.length"
        >
          Simpan Data
        </button>
        <button
          class="btn  text-black dark:text-white bg-red-500"
          @click="removeDataLocalStorage"
        >
          Hapus Data
        </button>
        <button v-if="abData.length" class="btn btn-success" @click="exportDataToExcel">
          Export to Excel
        </button>
      </div>

      <div class="w-full md:w-auto ">
        <div class="relative">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Search..."
            class="input input-bordered input-success w-full md:w-64 pl-10 text-black dark:text-white bg-white dark:bg-base-100"
            @input="debouncedSearch"
          />
          <span class="absolute left-3 top-1/2 transform -translate-y-1/2">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
              />
            </svg>
          </span>
        </div>
      </div>
    </div>

    <div v-if="isLoading" class="text-center py-8">
      <span class="loading loading-spinner loading-lg"></span>
      <p>Processing data...</p>
    </div>

    <div v-else-if="filteredData.length" class="overflow-x-auto text-black dark:text-white bg-white dark:bg-gray-700 rounded-lg shadow">
      <table class="table w-full text-black dark:text-white ">
        <thead>
          <tr>
            <th
              v-for="header in tableHeaders"
              :key="header"
              class="cursor-pointer text-black dark:text-white"
              @click="sortTable(header)"
            >
              <div class="flex items-center">
                {{ header }}
                <span v-if="sortColumn === header" class="ml-1">
                  {{ sortDirection === 'asc' ? '↑' : '↓' }}
                </span>
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in paginatedData" :key="index">
            <td v-for="header in tableHeaders" :key="header">
              {{ row[header] }}
            </td>
          </tr>
        </tbody>
      </table>

      <div class="flex flex-col md:flex-row justify-between items-center p-4 border-t gap-4 text-black dark:text-white bg-white dark:bg-base-100">
        <div class="flex items-center gap-4">
          <span class="text-sm ">Items per page:</span>
          <select
            v-model="pageSize"
            class="select select-bordered select-sm"
            @change="currentPage = 1"
          >
            <option value="10">10</option>
            <option value="25">25</option>
            <option value="50">50</option>
            <option value="100">100</option>
          </select>
        </div>

        <div class="text-sm ">
          Showing {{ pagination.startItem }} to {{ pagination.endItem }} of
          {{ pagination.totalItems }} entries
        </div>

        <div class="flex space-x-2">
          <button
            @click="changePage(currentPage - 1)"
            :disabled="currentPage === 1"
            class="btn btn-sm"
            :class="{ 'btn-disabled': currentPage === 1 }"
          >
            Previous
          </button>
          <button
            v-for="page in visiblePages"
            :key="page"
            @click="changePage(page)"
            class="btn btn-sm"
            :class="{ 'btn-active': currentPage === page }"
          >
            {{ page }}
          </button>
          <button
            @click="changePage(currentPage + 1)"
            :disabled="currentPage === pagination.totalPages"
            class="btn btn-sm"
            :class="{ 'btn-disabled': currentPage === pagination.totalPages }"
          >
            Next
          </button>
        </div>
      </div>
    </div>

    <div v-else class="text-center py-8 text-gray-500">
      {{
        abData.length
          ? 'No matching results found'
          : 'No data available. Please import an Excel file.'
      }}
    </div>
  </div>
</template>

<script setup>
import * as XLSX from 'xlsx'
import { ref, computed, onMounted, watch } from 'vue'
import { debounce } from 'lodash'

// Data refs
const excelData = ref([])
const dataExcel = ref([])
const groupedData = ref([])
const abData = ref([])
const tableHeaders = ref([])
const isLoading = ref(false)

// Pagination and search
const currentPage = ref(1)
const pageSize = ref(10)
const searchQuery = ref('')
const totalItems = ref(0)

// Sorting
const sortColumn = ref('')
const sortDirection = ref('asc')

const handleFileChange = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  isLoading.value = true
  try {
    await readExcelFile(file)
  } catch (error) {
    alert('Error reading file: ' + error.message)
  } finally {
    isLoading.value = false
  }
}

const readExcelFile = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = (e) => {
      try {
        const data = new Uint8Array(e.target.result)
        const workbook = XLSX.read(data, { type: 'array' })
        const sheetName = workbook.SheetNames[0]
        const sheet = workbook.Sheets[sheetName]
        excelData.value = XLSX.utils.sheet_to_json(sheet)
        resolve()
      } catch (error) {
        reject(error)
      }
    }
    reader.onerror = () => reject(new Error('File reading failed'))
    reader.readAsArrayBuffer(file)
  })
}

const saveDataToLocalstorage = () => {
  if (excelData.value.length) {
    localStorage.setItem('excelData', JSON.stringify(excelData.value))
    dataExcel.value = [...excelData.value]
    processDataForDisplay()
    alert('Data saved successfully')
  } else {
    alert('No data to save')
  }
}

const removeDataLocalStorage = () => {
  localStorage.removeItem('excelData')
  dataExcel.value = []
  processDataForDisplay()
  alert('Data removed successfully')
}


const formatWaktu = (datetime) => {
  if (!datetime || typeof datetime !== 'string') {
    return { FormattedDateTime: '', DaysPart: '', DatePart: '', TimePart: '' }
  }

  const daysOfWeek = ['Minggu', 'Senin', 'Selasa', 'Rabu', 'Kamis', "Jum'at", 'Sabtu']
  const [datePart, timePart] = datetime.split(' ')
  const [day, month, year] = datePart.split('/')

  try {
    const parsedDate = new Date(`${year}-${month}-${day}T${timePart}`)
    if (isNaN(parsedDate.getTime()))
      return { FormattedDateTime: '', DaysPart: '', DatePart: '', TimePart: '' }

    const dayOfWeek = daysOfWeek[parsedDate.getDay()]
    return {
      FormattedDateTime: `${dayOfWeek}, ${datePart}`,
      DaysPart: dayOfWeek,
      DatePart: datePart,
      TimePart: timePart,
    }
  } catch {
    return { FormattedDateTime: '', DaysPart: '', DatePart: '', TimePart: '' }
  }
}

const groupDataByNoID = () => {
  groupedData.value = dataExcel.value
    .filter((record) => record.Pengecualian !== 'Mengulang')
    .reduce((result, record) => {
      const noID = record['No.ID']
      const { DatePart, TimePart, DaysPart } = formatWaktu(record["Tgl/Waktu"])

      if (!result[noID]) {
        result[noID] = {
          'No.ID': noID,
          Department: record.Departemen,
          Nama: record.Nama,
          DateEntries: {},
        }
      }

      if (!result[noID].DateEntries[DatePart]) {
        result[noID].DateEntries[DatePart] = {
          DatePart,
          Entries: [],
          'Jam Masuk Kerja': '',
          'Jam Pulang Kerja': '',
          'Terlambat Masuk Kerja': '',
          // Lembur: '',
        }
      }


      const dateEntry = result[noID].DateEntries[DatePart]
      const status = record.Status

      // Check if we already have a C/Masuk entry for this date
      const hasMasukEntry = dateEntry.Entries.some(entry => entry.Status === 'C/Masuk')
      
      // Skip if this is a duplicate C/Masuk for the same date
      if (status === 'C/Masuk' && hasMasukEntry) {
        return result
      }
      
      const isLate = status === 'C/Masuk' && isLateArrival(TimePart)
      const isOvertime = (TimePart > '18:00' && status === 'C/Keluar') || DaysPart === 'Minggu'
      const isInvalid = record.Pengecualian === 'Invalid'

      if (isLate) dateEntry['Terlambat Masuk Kerja'] = 'Terlambat'
      if (isInvalid) dateEntry['Status Absen'] = 'Invalid'
      if (isOvertime) dateEntry['Lembur'] = 'Lembur'

      if (status === 'C/Masuk' || status === 'Lembur Masuk') {
        dateEntry['Jam Masuk Kerja'] = TimePart
      }

      if (status === 'C/Keluar' || status === 'Lembur Keluar') {
        dateEntry['Jam Pulang Kerja'] = TimePart
      }


      dateEntry.Entries.push({
        Tanggal: formatWaktu(record["Tgl/Waktu"]).FormattedDateTime,
        Jam: TimePart,
        Hari: DaysPart,
      })

      return result
    }, {})

  groupedData.value = Object.values(groupedData.value)
  flattenGroupedData()
}

const isLateArrival = (checkInTime) => {
  const [hours, minutes] = checkInTime.split(':').map(Number)
  return hours > 8 || (hours === 8 && minutes > 30)
}

const flattenGroupedData = () => {
  abData.value = groupedData.value.flatMap((item) =>
    Object.values(item.DateEntries).map((dateEntry) => {
      // Get the day from the first entry (since all entries for the same date have the same day)
      const hari = dateEntry.Entries[0]?.Hari || '-';
      
      return {
        'No.ID': item['No.ID'],
        Department: item.Department,
        Nama: item.Nama,
        Hari: hari,  // Add the day here
        Tanggal: dateEntry.DatePart,
        'Jam Masuk Kerja': dateEntry['Jam Masuk Kerja'] || '-',
        'Jam Pulang Kerja': dateEntry['Jam Pulang Kerja'] || '-',
        'Terlambat Masuk Kerja': dateEntry['Terlambat Masuk Kerja'] || '-',
        Lembur: dateEntry['Lembur'] || '-',
        // 'Status Absen': dateEntry['Status Absen'] || '-',
      };
    })
  );

  if (abData.value.length) {
    tableHeaders.value = Object.keys(abData.value[0]);
  }
};

const exportDataToExcel = () => {
  if (!abData.value.length) {
    alert('No data to export')
    return
  }

  const worksheet = XLSX.utils.json_to_sheet(sortedData.value)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, 'AttendanceData')
  XLSX.writeFile(workbook, `attendance_report_${new Date().toISOString().slice(0, 10)}.xlsx`)
}

// Pagination and search
const pagination = computed(() => ({
  startItem: (currentPage.value - 1) * pageSize.value + 1,
  endItem: Math.min(currentPage.value * pageSize.value, totalItems.value),
  totalItems: totalItems.value,
  totalPages: Math.ceil(totalItems.value / pageSize.value),
}))

const visiblePages = computed(() => {
  const range = 2
  let start = Math.max(1, currentPage.value - range)
  let end = Math.min(pagination.value.totalPages, currentPage.value + range)

  if (currentPage.value <= range + 1) {
    end = Math.min(2 * range + 1, pagination.value.totalPages)
  }
  if (currentPage.value >= pagination.value.totalPages - range) {
    start = Math.max(1, pagination.value.totalPages - 2 * range)
  }

  return Array.from({ length: end - start + 1 }, (_, i) => start + i)
})

const filteredData = computed(() => {
  const query = searchQuery.value.toLowerCase()
  return query
    ? abData.value.filter((item) =>
        Object.values(item).some((value) => String(value).toLowerCase().includes(query)),
      )
    : abData.value
})

const sortedData = computed(() => {
  if (!sortColumn.value) return filteredData.value

  return [...filteredData.value].sort((a, b) => {
    const valA = a[sortColumn.value]
    const valB = b[sortColumn.value]

    if (valA < valB) return sortDirection.value === 'asc' ? -1 : 1
    if (valA > valB) return sortDirection.value === 'asc' ? 1 : -1
    return 0
  })
})

const paginatedData = computed(() => {
  totalItems.value = sortedData.value.length
  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return sortedData.value.slice(start, end)
})

const debouncedSearch = debounce(() => {
  currentPage.value = 1
}, 300)

const changePage = (page) => {
  if (page >= 1 && page <= pagination.value.totalPages) {
    currentPage.value = page
  }
}

const sortTable = (column) => {
  if (sortColumn.value === column) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortColumn.value = column
    sortDirection.value = 'asc'
  }
}

const processDataForDisplay = () => {
  groupDataByNoID()
  currentPage.value = 1
  searchQuery.value = ''
  sortColumn.value = ''
  sortDirection.value = 'asc'
}

onMounted(() => {
  const storedData = localStorage.getItem('excelData')
  if (storedData) {
    dataExcel.value = JSON.parse(storedData)
    processDataForDisplay()
  }
})
</script>
