<template>
  <div class="app">
    <header>
      <h1>Справочник организаций</h1>
    </header>

    <!-- Поле поиска -->
    <div class="searchAndAdd">
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Найти по ФИО"
        class="search-input"
      />
      <button @click="openModal">Добавить</button>
    </div>

    <!-- Таблица с организациями -->
    <table>
      <thead>
        <tr>
          <th @click="sort('name')">
            Название организации<span v-if="sortKey === 'name'">{{
              sortOrder === "asc" ? "⬆️" : "⬇️"
            }}</span>
          </th>
          <th @click="sort('director')">
            ФИО директора<span v-if="sortKey === 'director'">{{
              sortOrder === "asc" ? "⬆️" : "⬇️"
            }}</span>
          </th>
          <th>Номер телефона</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="org in paginatedOrganizations" :key="org.id">
          <td>{{ org.name }}</td>
          <td>{{ org.director }}</td>
          <td>{{ org.phone }}</td>
          <td>
            <button @click="deleteOrganization(org.id)">❌</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Пагинация -->
    <div class="pagination">
      <button
        :disabled="currentPage === 1"
        @click="changePage(currentPage - 1)"
      >
        &lt;
      </button>
      <span>Страница {{ currentPage }} из {{ totalPages }}</span>
      <button
        :disabled="currentPage === totalPages"
        @click="changePage(currentPage + 1)"
      >
        &gt;
      </button>
    </div>
    <!-- Модальное окно -->
    <div v-if="isModalOpen" class="modal">
      <div class="modal-content">
        <h2>Добавить организацию</h2>
        <form @submit.prevent="addOrganization">
          <input
            class="input_modal"
            type="text"
            v-model="newOrganization.name"
            required
            placeholder="Название"
          />
          <input
            class="input_modal"
            type="text"
            v-model="newOrganization.phone"
            required
            placeholder="Номер телефона"
          />
          <input
            class="input_modal"
            type="text"
            v-model="newOrganization.director"
            required
            placeholder="ФИО директора"
          />
          <div class="modal-actions">
            <button type="button" @click="closeModal">Отмена</button>
            <button type="submit" :disabled="!isFormValid">ОК</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue";

export default {
  setup() {
    // Список базовых(тестовых) организаций
    const organizations = ref([
      {
        id: 1,
        name: "ООО Пятерочка",
        director: "Иванов И.В",
        phone: "89996661122",
      },
      {
        id: 2,
        name: "ООО Магнит",
        director: "Петров П.А",
        phone: "89998882211",
      },
    ]);

    // Управление состоянием
    const searchQuery = ref("");
    const sortKey = ref(null);
    const sortOrder = ref(null);
    const currentPage = ref(1);
    const pageSize = ref(5);
    const isModalOpen = ref(false);

    // Новая организация
    const newOrganization = ref({
      name: "",
      director: "",
      phone: "",
    });

    // Фильтрация и сортировка
    const filteredOrganizations = computed(() => {
      return organizations.value.filter((org) =>
        org.director.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    const sortedOrganizations = computed(() => {
      if (!sortKey.value) return filteredOrganizations.value;
      return [...filteredOrganizations.value].sort((a, b) => {
        const order = sortOrder.value === "asc" ? 1 : -1;
        return a[sortKey.value].localeCompare(b[sortKey.value]) * order;
      });
    });

    const paginatedOrganizations = computed(() => {
      const start = (currentPage.value - 1) * pageSize.value;
      return sortedOrganizations.value.slice(start, start + pageSize.value);
    });

    const totalPages = computed(() => {
      return Math.ceil(filteredOrganizations.value.length / pageSize.value);
    });

    // Методы
    const sort = (key) => {
      if (sortKey.value === key) {
        sortOrder.value = sortOrder.value === "asc" ? "desc" : "asc";
      } else {
        sortKey.value = key;
        sortOrder.value = "asc";
      }
    };

    const changePage = (page) => {
      if (page >= 1 && page <= totalPages.value) {
        currentPage.value = page;
      }
    };

    const deleteOrganization = (id) => {
      organizations.value = organizations.value.filter((org) => org.id !== id);
    };

    const openModal = () => {
      isModalOpen.value = true;
    };

    const closeModal = () => {
      isModalOpen.value = false;
      resetNewOrganization();
    };

    const addOrganization = () => {
      organizations.value.push({
        id: Date.now(),
        ...newOrganization.value,
      });
      closeModal();
    };

    const resetNewOrganization = () => {
      newOrganization.value = {
        name: "",
        director: "",
        phone: "",
      };
    };

    const isFormValid = computed(() => {
      return (
        newOrganization.value.name &&
        newOrganization.value.director &&
        newOrganization.value.phone
      );
    });

    return {
      organizations,
      searchQuery,
      sortKey,
      sortOrder,
      currentPage,
      pageSize,
      isModalOpen,
      newOrganization,
      filteredOrganizations,
      sortedOrganizations,
      paginatedOrganizations,
      totalPages,
      sort,
      changePage,
      deleteOrganization,
      openModal,
      closeModal,
      addOrganization,
      isFormValid,
    };
  },
};
</script>

<style scoped lang="scss">
.app {
  max-width: 800px;
  margin: 0 auto;
  box-sizing: border-box;
  padding: 20px;
  font-size: 1em;
}
header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}
.searchAndAdd {
  display: flex;
  flex-direction: row;
  gap: 3em;
  .search-input {
    font-size: 1rem;
    width: auto;
    padding: 10px;
    margin-bottom: 20px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  button {
    margin-bottom: 20px;
    padding: 1em 1em;
    &:hover {
      background-color: #2d7069;
      color: #fff;
    }
  }
}
table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
  th,
  td {
    border: 1px solid #ccc;
    padding: 8px;
    text-align: left;
  }
  th {
    cursor: pointer;
  }
}
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}
button {
  padding: 2px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  .modal-content {
    background: white;
    padding: 20px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    width: 300px;
    padding: 20px;
    h2 {
      text-align: center;
    }
    form {
      display: flex;
      flex-direction: column;
      .input_modal {
        width: calc(100% - 20px);
        margin: 10px 0;
        padding: 10px;
      }
      .modal-actions {
        display: flex;
        justify-content: space-around;
        margin-top: 20px;
        button {
          padding: 10px 20px;
          &:hover {
            background-color: #2d7069;
            color: #fff;
          }
        }
      }
    }
  }
}
</style>
