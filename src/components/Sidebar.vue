<script setup lang="ts">
import { useStore } from "vuex";
import { computed, ComputedRef, onMounted, ref, Ref, watch } from "vue";
import { AddressModel } from "../models/address.model";
import AddressPreview from "./PreviewItem.vue";

const numShownAddresses: Ref<number> = ref(10);
const previewItemsContainerRef: Ref<HTMLDivElement | null> = ref(null);

const store = useStore();
const selectedAddress: ComputedRef<AddressModel | null> = computed(
  () => store.getters.getSelectedItem
);

const pageStartElemIdx: ComputedRef<number> = computed(
  () => store.getters["pagination/getStartElemIdx"]
);
const pageEndElemIdx: ComputedRef<number> = computed(
  () => store.getters["pagination/getEndElemIdx"]
);
const shownAddresses: ComputedRef<AddressModel[]> = computed(
  () => store.getters["map/getShownAddresses"]
);

onMounted(() => {
  console.log(shownAddresses); //store.getters["map/getShownAddresses"])

  watch(
    () => store.getters["map/getShownAddresses"],
    () => {
      // console.log("Showing new addresses, resetting scroll.");
      previewItemsContainerRef.value?.scrollTo({ top: 0, behavior: "smooth" });
      numShownAddresses.value = 10;
    }
  );
});

const getNumberOfDocuments = (): number => {
  const shownDocumentsNums: number[] = shownAddresses.value.map(
    (address: AddressModel) => address.documents.length
  );
  return shownDocumentsNums.reduce((prev, curr) => prev + curr, 0);
};

const onScroll = (e) => {
  const { scrollTop, offsetHeight, scrollHeight } = e.target;
  const hasScrolledToBottom = scrollTop + offsetHeight >= scrollHeight;
  if (hasScrolledToBottom) {
    numShownAddresses.value += 10;
  }
};
</script>

<template>
  <div
    class="md:col-span-2 bg-slate-600 py-0 overflow-y-auto h-[50vh] md:h-full text-white"
    id="preview-items-container"
    ref="previewItemsContainerRef"
    @scroll="onScroll($event)"
  >
    <h1
      class="text-lg mb-4 top-0 px-4 py-4 drop-shadow-2xl bg-slate-700 z-10"
      v-if="!selectedAddress"
    >
      <strong>Totaal:</strong>
      {{ shownAddresses.length }} adres{{
        shownAddresses.length !== 1 ? "sen" : ""
      }}
      en {{ getNumberOfDocuments() }} document{{
        getNumberOfDocuments() !== 1 ? "en" : ""
      }}
    </h1>
    <div class="px-4 pt-4">
      <template v-if="selectedAddress">
        <address-preview :address="selectedAddress"></address-preview>
      </template>
      <template v-if="!selectedAddress">
        <address-preview
          :address="shownAddress"
          v-for="shownAddress in shownAddresses.slice(0, numShownAddresses)"
        ></address-preview>
      </template>
    </div>
  </div>
</template>