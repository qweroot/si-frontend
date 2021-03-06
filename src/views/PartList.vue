<template>
  <div>
    <div class="flex items-center justify-between px-2">
      <div>
        <Title title="Part List" />
      </div>
      <div>
        <Button @click="startEditItem(defaultItem, createFunction)">
          <icon :d="icons.plus" class="h-5 w-5 inline-block" />
          부품
        </Button>
      </div>
    </div>
    <div class="h-auto shadow-2xs overflow-x-auto rounded-lg relative">
      <DataTable :headers="headers" :data="items" selectable>
        <template v-slot:col_quantity="{ item }">
          <div class="flex items-center">
            <div class="w-2 h-2 mx-2 bg-blue-300 rounded-full"></div>
            {{ item.quantity }}
          </div>
        </template>

        <template v-slot:col_category="{ item }">
          {{ getCategoryNameFromId(item.category) }}
        </template>

        <template v-slot:col_action="{ item }">
          <div class="flex items-center">
            <Button @click="startEditItem(item, editFunction)" text>
              <icon :d="icons.pencilAlt" class="h-4 w-4" />
            </Button>

            <Button
              @click="startDeleteItem(item, deleteFunction)"
              text
              type="danger"
            >
              <icon :d="icons.trash" class="h-4 w-4" />
            </Button>
          </div>
        </template>
      </DataTable>
    </div>
    <Slider>
      <template v-slot:header>
        <div class="flex items-center justify-between">
          <div>
            This is custom header
          </div>
          <div>
            <Button @click="createPartFromFile" text>파일로 추가</Button>
          </div>
        </div>
      </template>

      <template v-slot:content>
        <div v-for="(header, i) in editHeaders" :key="i" class="my-2">
          <labeled-input
            v-if="header.type === 'text'"
            :label="header.text"
            v-model="editedItem[header.value]"
          />
          <template v-else-if="header.type === 'select'">
            <label class="block text-sm font-medium text-gray-700"
              >Category
            </label>
            <dropdown
              v-model="editedItem.category"
              :items="categories"
              item-value="id"
              item-text="name"
              class="mt-1"
            />
          </template>
        </div>
        <div class="flex items-center justify-between px-2">
          <div class="flex-1 text-right">
            <Button @click="processEditItem"> {{ saveButtonLabel }} </Button>
          </div>
        </div>
      </template>
    </Slider>
    <Modal>
      <template v-slot:body>
        <input
          id="file-upload"
          name="file-upload"
          type="file"
          @change="handleFileSelect"
        />
      </template>

      <template v-slot:action>
        <Button @click="handleFileUpload">Save</Button>
      </template>
    </Modal>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, defineComponent, computed } from "vue";
import { IPart, ICategory } from "@/types/inventory";
import { useItemCrud } from "@/components/hooks/useItemCrud";
import { useOpenState } from "@/components/hooks/useOpenState";
import { usePagination } from "@/components/hooks/usePagination";
import Title from "@/components/base/Title.vue";
import DataTable from "@/components/table/DataTable.vue";
import Slider from "@/components/slider/Slider.vue";
import partService from "@/services/partService";
import CategoryService from "@/services/categoryService";
import Button from "@/components/button/Button.vue";
import Dropdown from "@/components/dropdown/Dropdown.vue";
import Icon from "@/components/icon/Icon.vue";
import Modal from "@/components/base/Modal.vue";
import LabeledInput from "@/components/input/LabeledInput.vue";
import { useRoute } from "vue-router";

export default defineComponent({
  components: {
    Button,
    DataTable,
    Dropdown,
    Icon,
    Modal,
    Slider,
    Title,
    LabeledInput,
  },
  setup() {
    const route = useRoute()
    const icons = require("@/assets/icons.json");
    const openCategoryDropdown = ref(false);
    const deleteFunction = ref(partService.deletePart);
    const createFunction = ref(partService.createPart);
    const editFunction = ref(partService.updatePart);
    const categories = ref([]);
    const file = ref(new Blob());
    const { totalCount } = usePagination();
    const { isOpenModal, isOpenSlider } = useOpenState();
    const {
      editedIndex,
      editedItem,
      items,
      processEditItem,
      processDeleteItem,
      startEditItem,
      startDeleteItem,
    } = useItemCrud();
    const defaultItem = {
      name: "",
      spec: "",
      serial: "",
      desc: "",
      category: 1,
      quantity: 0,
      ideal_quantity: 0,
      order_quantity: 0,
    };
    const headers = ref([
      { text: "Name", value: "name", type: "text" },
      { text: "Description", value: "desc", type: "text" },
      { text: "Category", value: "category", type: "select" },
      { text: "Spec", value: "spec", type: "text" },
      { text: "Serial", value: "serial", type: "text" },
      { text: "Quantity", value: "quantity", type: "text" },
      { text: "Ideal quantity", value: "ideal_quantity", type: "text" },
      { text: "Order quantity", value: "order_quantity", type: "text" },
      { text: "", value: "action" },
    ]);

    const editHeaders = computed(() => {
      return headers.value.filter((header) => {
        return header.value !== "action";
      });
    });

    const saveButtonLabel = computed(() => {
      return editedIndex.value > -1 ? "수정" : "추가";
    });

    function fetchData(params: any) {
      partService.getParts(params).then(
        (data: IPart) => {
          items.value = data;
          totalCount.value = items.value.length;
        },
        (err: any) => {
          console.log(err);
        }
      );
    }

    function fetchCategories() {
      CategoryService.getCategories({}).then(
        (data) => {
          categories.value = data;
        },
        (error) => {
          console.log(error);
        }
      );
    }

    function setCategory(category: any) {
      openCategoryDropdown.value = false;
      editedItem.value.category = category.id;
    }

    function getCategoryNameFromId(id: number) {
      const category = categories.value.find(
        (category) => (category as ICategory).id === id
      );
      if (category !== undefined) {
        return (category as ICategory).name;
      }
      return "Categories";
    }

    function createPartFromFile() {
      isOpenSlider.value = false;
      isOpenModal.value = true;
    }

    function handleFileSelect(e: any) {
      console.log(e.target.files[0]);
      file.value = e.target.files[0];
    }

    function handleFileUpload() {
      isOpenModal.value = false;
      if (file.value) {
        const formData = new FormData();
        formData.append("file", file.value);

        partService.createPartBulk(formData).then(
          () => {
            console.log("ok");
          },
          (err: any) => {
            console.log(err);
          }
        );
      }
    }

    onMounted(() => {
      const query = route.query;
      fetchData(query);
      fetchCategories();
    });

    return {
      categories,
      defaultItem,
      editedItem,
      editedIndex,
      editHeaders,
      file,
      getCategoryNameFromId,
      handleFileSelect,
      handleFileUpload,
      createPartFromFile,
      headers,
      icons,
      items,
      isOpenModal,
      openCategoryDropdown,
      saveButtonLabel,
      setCategory,
      startEditItem,
      startDeleteItem,
      processEditItem,
      processDeleteItem,
      deleteFunction,
      createFunction,
      editFunction,
    };
  },
});
</script>

<style scoped></style>
