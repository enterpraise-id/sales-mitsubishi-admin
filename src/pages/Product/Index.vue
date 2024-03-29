<script setup lang="ts">
import { useDialog } from "naive-ui";
import { reactive } from "vue";
import useDeleteProduct from "../../composable/useDeleteProduct";
import useGetProducts from "../../composable/useGetProducts";
import CreateProduct from "./components/CreateProduct.vue";
import ViewProduct from "./components/ViewProduct.vue";
import EditProduct from "./components/EditProduct.vue";
import { Product } from "../../typings/Product";

interface Modal {
  create: {
    open: boolean;
  };
  view: {
    open: boolean;
    item: Partial<Product>;
  };
  edit: {
    open: boolean;
    item: Product | null;
  };
}

const modal = reactive<Modal>({
  create: {
    open: false,
  },
  view: {
    open: false,
    item: {},
  },
  edit: {
    open: false,
    item: null,
  },
});

const { data: products, refetch } = useGetProducts();
const { mutate: deleteProduct, onSuccess: onDeleteSuccess } = useDeleteProduct();
const dialog = useDialog();

const onViewButtonClick = (item: any) => {
  modal.view.open = true;
  modal.view.item = item;
};

const onEditButtonClick = (item: any) => {
  modal.edit.open = true;
  modal.edit.item = item;
};

const onButtonDeleteClick = (id: number) => {
  dialog.warning({
    title: "Hapus Produk",
    content: "Apakah anda yakin ingin menghapus produk ini?",
    positiveText: "Yakin",
    negativeText: "Tidak",
    onPositiveClick: () => {
      deleteProduct(id);
    },
  });
};

onDeleteSuccess(() => {
  refetch();
});
</script>

<template>
  <n-card>
    <n-button type="primary" @click="modal.create.open = true">Tambah Produk</n-button>
    <n-table class="mt-30">
      <thead>
        <tr>
          <th>Nama</th>
          <th>Jumlah Tipe</th>
          <th>#Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in products">
          <td>{{ item.name }}</td>
          <td>{{ item.types.length }}</td>
          <td class="w-100">
            <n-button-group>
              <n-button type="primary" @click="onViewButtonClick(item)">Lihat</n-button>
              <n-button type="info" @click="onEditButtonClick(item)">Edit</n-button>
              <n-button type="error" @click="onButtonDeleteClick(item.id)"
                >Hapus</n-button
              >
            </n-button-group>
          </td>
        </tr>
      </tbody>
    </n-table>
  </n-card>

  <CreateProduct
    :open="modal.create.open"
    @close="modal.create.open = false"
    @created="refetch"
  />
  <ViewProduct
    :open="modal.view.open"
    :item="modal.view.item"
    @close="modal.view.open = false"
  />
  <EditProduct
    :open="modal.edit.open"
    :item="modal.edit.item"
    @close="modal.edit.open = false"
    @updated="refetch"
  />
</template>

<style scoped>
.mt-30 {
  margin-top: 30px;
}
.w-100 {
  width: 100px;
}
</style>
