<script setup lang="ts">
import { ref, onMounted, computed, watch } from "vue";
import BreadcrumbDefault from "@/components/Breadcrumbs/BreadcrumbDefault.vue";
import DefaultLayout from "@/layouts/DefaultLayout.vue";
import { adminTeknis_GetDataEvidentByType } from "@/stores/functionAPI";
import {
  mdiEyeOutline,
  mdiMagnify,
  mdiRefresh,
  mdiPlusCircleOutline,
  mdiSquareEditOutline,
} from "@mdi/js";
import { useIndexStore } from "@/stores";

const indexStore = useIndexStore();
const pageTitle = ref("Evident - PSN");
const pageList = ref(["Work Order", "Evident", "PSN"]);
const dataHeader = ref([
  { name: "No.", class: "py-2 pl-3" },
  { name: "Tgl. Dibuat", class: "min-w-[100px] py-2 px-4" },
  { name: "No. Logistik", class: "min-w-[150px] py-2 px-4" },
  { name: "Id. User", class: "min-w-[100px] py-2 px-4" },
  { name: "Nama. User", class: "min-w-[150px] py-2 px-4" },
  { name: "Server", class: "py-2 px-4" },
  { name: "Kategori", class: "py-2 px-4" },
  { name: "Tr Teknis", class: "py-2 px-4" },
]);
let dataTable = ref([]);
const searchQuery = ref("");

// Pagination state
const currentPage = ref(1);
const itemsPerPageOptions = [5, 10, 20, 50]; // Pilihan jumlah data per halaman
const itemsPerPage = ref(5);
const pageInput = ref(currentPage.value);

watch(itemsPerPage, () => {
  currentPage.value = 1; // Reset to the first page
});

const jumpToPage = () => {
  if (pageInput.value >= 1 && pageInput.value <= totalPages.value) {
    changePage(pageInput.value);
  } else {
    // Reset to the current page if input is invalid
    pageInput.value = currentPage.value;
  }
};

// Filtered data based on search query
const filteredData = computed(() => {
  if (!searchQuery.value.trim()) return dataTable.value;
  return dataTable.value.filter((item) =>
    Object.values(item).join(" ").toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// Total items based on the filtered data
const totalItems = computed(() => filteredData.value.length);
const totalPages = computed(() => Math.ceil(totalItems.value / itemsPerPage.value));

// Paginated data based on filtered results
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return filteredData.value.slice(start, end);
});

// Change page function
const changePage = (page: number) => {
  if (page > 0 && page <= totalPages.value) {
    currentPage.value = page;
  }
};

const visiblePages = computed(() => {
  const pages = [];
  const range = 2; // Number of pages to show before and after the current page

  let start = Math.max(1, currentPage.value - range);
  let end = Math.min(totalPages.value, currentPage.value + range);

  for (let i = start; i <= end; i++) {
    pages.push(i);
  }

  return pages;
});

// Change page function
const getMainData = async () => {
  const data = await adminTeknis_GetDataEvidentByType(
    "N",
    "PSN",
    indexStore.company.companyCode,
    indexStore.user.hierarchyCode
  );
  dataTable.value = data;
};

onMounted(async () => {
  await getMainData();
});
</script>

<template>
  <DefaultLayout>
    <BreadcrumbDefault :pageTitle="pageTitle" :pageList="pageList" />

    <div class="flex flex-col gap-10">
      <div
        class="rounded-sm border border-stroke bg-white px-5 py-6 shadow-default dark:border-strokedark dark:bg-boxdark sm:px-7.5"
      >
        <div
          class="helper border-2 flex justify-between bg-gray-2 text-left dark:bg-meta-4 text-black dark:text-white py-2 px-4"
        >
          <div class="left-data">
            <div class="relative">
              <button class="absolute top-1/2 left-0 -translate-y-1/2">
                <svg
                  class="fill-body hover:fill-primary dark:fill-bodydark dark:hover:fill-primary"
                  width="20"
                  height="20"
                  viewBox="0 0 20 20"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path :d="mdiMagnify" fill="" />
                </svg>
              </button>

              <input
                type="text"
                placeholder="Type to search..."
                class="w-full xl:w-125 bg-transparent pr-4 pl-9 focus:outline-none"
                v-model="searchQuery"
              />
            </div>
          </div>
          <div class="right-data flex items-center flex-row-reverse">
            <router-link to="/modules/work-order/evident/psn/add" class="px-1">
              <svg
                class="fill-current hover:text-primary"
                width="22"
                height="22"
                viewBox="0 0 24 24"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path :d="mdiPlusCircleOutline" fill="" />
              </svg>
            </router-link>
            <button class="px-1" @click="getMainData()">
              <svg
                class="fill-current hover:text-primary"
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path :d="mdiRefresh" fill="" />
              </svg>
            </button>
          </div>
        </div>

        <div class="max-w-full overflow-x-auto">
          <table class="w-full table-auto">
            <thead>
              <tr class="bg-gray-2 text-left dark:bg-meta-4">
                <th
                  v-for="data in dataHeader"
                  :class="data.class"
                  class="font-medium text-black dark:text-white text-center text-sm"
                >
                  {{ data.name }}
                </th>
                <th class="py-2 px-4 font-medium text-black dark:text-white text-center">
                  Opsi
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in paginatedData" :key="index" class="border">
                <td class="pl-3 text-xs border">
                  {{ index + 1 }}
                </td>
                <td class="py-1 px-4 border">
                  <p class="text-black dark:text-white text-xs text-center">
                    {{ item.Tr_teknis_created }}
                  </p>
                </td>
                <td class="py-1 px-4 border">
                  <p class="text-black dark:text-white text-xs text-center">
                    {{ item.Tr_teknis_logistik_id }}
                  </p>
                </td>
                <td class="py-1 px-4 border">
                  <p class="text-xs text-black dark:text-white">
                    {{ item.Tr_teknis_pelanggan_id }}
                  </p>
                </td>
                <td class="py-1 px-4 border">
                  <h5 class="font-medium text-black text-xs dark:text-white">
                    {{ item.Tr_teknis_pelanggan_nama }}
                  </h5>
                </td>
                <td class="py-1 px-4 text-center border">
                  <h5 class="font-medium text-black text-xs dark:text-white">
                    {{ item.Tr_teknis_pelanggan_server }}
                  </h5>
                </td>
                <td class="py-1 px-4 text-center border">
                  <h5 class="font-medium text-black text-xs dark:text-white">
                    {{ item.Tr_teknis_kategori }}
                  </h5>
                </td>
                <td class="py-1 px-4 border">
                  <h5 class="font-medium text-black text-xs dark:text-white">
                    {{ item.Tr_teknis_user_updated }}
                  </h5>
                </td>
                <td class="py-1 px-4">
                  <div class="flex items-center space-x-3.5 d-flex justify-center">
                    <router-link
                      class="hover:text-primary"
                      :to="
                        '/modules/work-order/evident/psn/detail/' +
                        item.Tr_teknis_logistik_id +
                        '/' +
                        item._id
                      "
                    >
                      <svg
                        class="fill-current"
                        width="24"
                        height="24"
                        viewBox="0 0 24 24"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                      >
                        <path :d="mdiEyeOutline" fill="" />
                      </svg>
                    </router-link>
                    <router-link
                      class="hover:text-primary"
                      :to="
                        '/modules/work-order/evident/psn/edit/' +
                        item.Tr_teknis_logistik_id +
                        '/' +
                        item._id
                      "
                    >
                      <svg
                        class="fill-current"
                        width="24"
                        height="24"
                        viewBox="0 0 24 24"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                      >
                        <path :d="mdiSquareEditOutline" fill="" />
                      </svg>
                    </router-link>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="mt-4 flex justify-between items-center">
          <!-- Menampilkan Data -->
          <div class="flex items-center text-sm">
            <label for="itemsPerPage" class="mr-2">Menampilkan</label>
            <select
              id="itemsPerPage"
              v-model="itemsPerPage"
              class="px-2 py-1 border border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600 dark:text-white"
            >
              <option v-for="option in itemsPerPageOptions" :key="option" :value="option">
                {{ option }}
              </option>
            </select>
            <span class="ml-2">dari {{ totalItems }} data</span>
          </div>

          <!-- Navigasi Pagination -->
          <div class="flex space-x-2 items-center">
            <!-- Previous Button -->
            <button
              class="text-xs px-2 py-1 bg-gray-200 hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600"
              :disabled="currentPage === 1"
              @click="changePage(currentPage - 1)"
            >
              Prev
            </button>

            <!-- First Page -->
            <button
              v-if="currentPage > 3"
              class="text-xs px-2 py-1 bg-gray-200 hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600"
              @click="changePage(1)"
            >
              1
            </button>

            <!-- Ellipsis Before -->
            <span v-if="currentPage > 4" class="text-xs px-2 py-1">...</span>

            <!-- Visible Page Buttons -->
            <button
              v-for="page in visiblePages"
              :key="page"
              class="text-xs px-2 py-1 bg-gray-200 hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600"
              :class="currentPage === page ? 'bg-primary text-white' : ''"
              @click="changePage(page)"
            >
              {{ page }}
            </button>

            <!-- Ellipsis After -->
            <span v-if="currentPage < totalPages - 3" class="text-xs px-2 py-1">...</span>

            <!-- Last Page -->
            <button
              v-if="currentPage < totalPages - 2"
              class="text-xs px-2 py-1 bg-gray-200 hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600"
              @click="changePage(totalPages)"
            >
              {{ totalPages }}
            </button>

            <!-- Next Button -->
            <button
              class="text-xs px-2 py-1 bg-gray-200 hover:bg-gray-300 dark:bg-gray-700 dark:hover:bg-gray-600"
              :disabled="currentPage === totalPages"
              @click="changePage(currentPage + 1)"
            >
              Next
            </button>

            <!-- Page Number Input -->
            <div class="flex items-center ml-4">
              <label for="pageInput" class="mr-2">Go to:</label>
              <input
                id="pageInput"
                type="number"
                v-model.number="pageInput"
                @keyup.enter="jumpToPage"
                min="1"
                :max="totalPages"
                class="w-16 px-2 py-1 text-center border border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600 dark:text-white"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </DefaultLayout>
</template>
