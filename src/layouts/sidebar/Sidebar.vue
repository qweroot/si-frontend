<template>
  <div class="flex">
    <!-- Backdrop -->
    <div
      :class="isOpenSidebar ? 'block' : 'hidden'"
      @click="isOpenSidebar = false"
      class="fixed z-20 inset-0 bg-black opacity-50 transition-opacity lg:hidden"
    ></div>
    <!-- End Backdrop -->

    <div
      :class="
        isOpenSidebar ? 'translate-x-0 ease-out' : '-translate-x-full ease-in'
      "
      class="fixed z-30 inset-y-0 left-0 w-64 transition duration-300 transform bg-indigo-800 shadow-2xs overflow-y-auto lg:translate-x-0 lg:static lg:inset-0"
    >
      <div class="flex items-center justify-center mt-8">
        <div class="flex items-center">
          <svg
            class="h-12 w-12"
            viewBox="0 0 512 512"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M364.61 390.213C304.625 450.196 207.37 450.196 147.386 390.213C117.394 360.22 102.398 320.911 102.398 281.6C102.398 242.291 117.394 202.981 147.386 172.989C147.386 230.4 153.6 281.6 230.4 307.2C230.4 256 256 102.4 294.4 76.7999C320 128 334.618 142.997 364.608 172.989C394.601 202.981 409.597 242.291 409.597 281.6C409.597 320.911 394.601 360.22 364.61 390.213Z"
              fill="#4C51BF"
              stroke="#4C51BF"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
            <path
              d="M201.694 387.105C231.686 417.098 280.312 417.098 310.305 387.105C325.301 372.109 332.8 352.456 332.8 332.8C332.8 313.144 325.301 293.491 310.305 278.495C295.309 263.498 288 256 275.2 230.4C256 243.2 243.201 320 243.201 345.6C201.694 345.6 179.2 332.8 179.2 332.8C179.2 352.456 186.698 372.109 201.694 387.105Z"
              fill="white"
            />
          </svg>

          <span class="text-white text-2xl mx-2 font-semibold"
            >생산관리시스템</span
          >
        </div>
      </div>

      <nav class="mt-10">
        <router-link
          v-for="(link, i) in navLinks"
          :key="i"
          :class="[$route.name === link.name ? activeClass : inactiveClass]"
          class="flex items-center mt-1 mx-4 py-2 px-6 rounded-lg"
          :to="link.path"
        >
          <svg
            class="h-5 w-5"
            viewBox="0 0 20 20"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M5 3C3.89543 3 3 3.89543 3 5V7C3 8.10457 3.89543 9 5 9H7C8.10457 9 9 8.10457 9 7V5C9 3.89543 8.10457 3 7 3H5Z"
              fill="currentColor"
            />
            <path
              d="M5 11C3.89543 11 3 11.8954 3 13V15C3 16.1046 3.89543 17 5 17H7C8.10457 17 9 16.1046 9 15V13C9 11.8954 8.10457 11 7 11H5Z"
              fill="currentColor"
            />
            <path
              d="M11 5C11 3.89543 11.8954 3 13 3H15C16.1046 3 17 3.89543 17 5V7C17 8.10457 16.1046 9 15 9H13C11.8954 9 11 8.10457 11 7V5Z"
              fill="currentColor"
            />
            <path
              d="M11 13C11 11.8954 11.8954 11 13 11H15C16.1046 11 17 11.8954 17 13V15C17 16.1046 16.1046 17 15 17H13C11.8954 17 11 16.1046 11 15V13Z"
              fill="currentColor"
            />
          </svg>
          <span class="mx-4">{{ link.name }}</span>
        </router-link>
      </nav>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import { useOpenState } from "@/components/hooks/useOpenState";

export default defineComponent({
  setup() {
    const { isOpenSidebar } = useOpenState();
    const activeClass = ref(
      // "bg-gray-600 bg-opacity-25 text-gray-100 border-gray-100"
      "bg-white text-gray-700 border-gray-100 shadow-lg"
    );
    const inactiveClass = ref(
      "text-gray-500 hover:bg-gray-300 hover:bg-opacity-25 hover:text-gray-100"
    );
    const navLinks = ref([
      {
        path: "/dashboard",
        name: "Dashboard",
      },
      {
        path: "/category",
        name: "CategoryList",
      },
      {
        path: "/part",
        name: "PartList",
      },
      {
        path: "/product",
        name: "ProductList",
      },
      {
        path: "/order",
        name: "OrderList",
      },
      {
        path: "/task",
        name: "TaskList",
      },
    ]);
    return {
      isOpenSidebar,
      activeClass,
      inactiveClass,
      navLinks,
    };
  },
});
</script>
