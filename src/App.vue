<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from "vue";
import CardWarehouse from "./components/CardWarehouse.vue";
import axios from "axios";
import {
  Col as ACol,
  ConfigProvider as AConfigProvider,
  Form as AForm,
  FormItem as AFormItem,
  Row as ARow,
  Select as ASelect,
} from "ant-design-vue";
import ruRu from "ant-design-vue/es/locale/ru_RU";
import dayjs from "dayjs";
import ru from "dayjs/locale/ru";
import utc from "dayjs/plugin/utc";

dayjs.locale(ru);
dayjs.extend(utc);

let timerId = null;

onMounted(() => {
  initValues();
  timerId = setInterval(start, 11500); // Вызывать функцию каждые 11 секунд
});

onUnmounted(() => {
  clearInterval(timerId);
});

function start() {
  coefficientsGet();
}

const companyArray = [
  {
    id: 1,
    name: "ARANTA Decor",
    apiToken: "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjUwNDE3djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc2MjgwMDg0MSwiaWQiOiIwMTk2YzM0NS05ZTY2LTc0MDUtYjJmNC1iNmEzOWYxYjZhOGYiLCJpaWQiOjUzNzE1MjI2LCJvaWQiOjU3MDAwOCwicyI6MTAyNCwic2lkIjoiNGU5MzNjNTQtZGMwNi00MzlhLWEwNjYtZjVkZGYxZTA3YTQ1IiwidCI6ZmFsc2UsInVpZCI6NTM3MTUyMjZ9.3QPyyRRU8JyJtPLWU2waHMCDKsEVsjJoAcwokGM_v9gb4qe_-9rE5c7q0Z5QXNAphLCSHzdKo1AAobyQYw9k-w",
    telegramToken: "7397979520:AAFH3aY5u-PO9OOegXDl_5hvv1PLUp_3eQ4",
    chatId: "-4587468459",
  },
  {
    id: 2,
    name: "Sunflowers",
    apiToken: "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjUwNDE3djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc2MjgwMDgyNiwiaWQiOiIwMTk2YzM0NS02MmI2LTcyMmQtYjIwMi01OTQ2OGI2ZmI3M2YiLCJpaWQiOjk2OTgyNDY4LCJvaWQiOjQwNzg0NjMsInMiOjEwMjQsInNpZCI6IjBmMzY0NjAzLWE4MWMtNDNhZC05MjliLTJhZjMxOWFhZTczYyIsInQiOmZhbHNlLCJ1aWQiOjk2OTgyNDY4fQ.sQ-Vipbsdo6CS0NQ9UYEvbrFI0kv9fwpRKPffW39V9o8M0S7edh4Y9v5cIhCSuvdwEfmkbCFLhpK_cPwRf2KTQ",
    telegramToken: "6584534762:AAFtICID-IRygrU-pt_A_N7dA5ypkAwqCO8",
    chatId: "-4585796755",
  },
  {
    id: 4,
    name: "Ne Vi",
    apiToken: "eyJhbGciOiJFUzI1NiIsImtpZCI6IjIwMjUwNDE3djEiLCJ0eXAiOiJKV1QifQ.eyJlbnQiOjEsImV4cCI6MTc2MjgwMTI0MiwiaWQiOiIwMTk2YzM0Yi1iZDMyLTdjMDMtYjk2Yy1kODVhNmRlNzliNzkiLCJpaWQiOjE5NjI0NzM2LCJvaWQiOjQxMjc0NjcsInMiOjEwMjQsInNpZCI6Ijg0YjlkNmQzLTAxMTItNDBiZi05MTZiLWVlZDFkOGY3NjBhNSIsInQiOmZhbHNlLCJ1aWQiOjE5NjI0NzM2fQ.WkF5uw__G_ZtezJJwJVIAbN71D9DJKBgecA8cz0mDyOvTxfwk0-8X4vjoocWB5tFmvS7B4a6NptZosP4gc1CuA",
    telegramToken: "7529315847:AAGkdEUV-4uZvsauYev3WAA06YxvGUuYsd0",
    chatId: "-4562130542",
  },
];

const companyOptions = ref([]);
const companySelected = ref(JSON.stringify(companyArray[0]));

watch(companySelected, (newValue, oldValue) => {
  localStorage.setItem("company_selected_limit", JSON.stringify(newValue));

  if (newValue !== oldValue) {
    initValues();
  }
});

const transformedCompanyOptions = computed(() => {
  return companyArray.map(company => ({
    label: company.name,
    value: JSON.stringify(company)
  }));
});

companyOptions.value = transformedCompanyOptions.value;

const transformedCompanySelected = computed(() => JSON.parse(companySelected.value));

function fieldCompanies(fieldName) {
  return `${transformedCompanySelected.value.name.replaceAll(" ", "")}_${fieldName}`
}

function initValues() {
  const getCompanySelected = localStorage.getItem("company_selected_limit");
  companySelected.value = JSON.parse(getCompanySelected) || JSON.stringify(companyArray[0]);

  const getWarehousesSelect = localStorage.getItem(fieldCompanies("warehouses_selected_limit"));
  warehousesSelected.value = getWarehousesSelect ? JSON.parse(getWarehousesSelect) : [];

  const getWarehouses = localStorage.getItem(fieldCompanies("warehouses_limit"));

  const resultWarehouses = getWarehouses ? JSON.parse(getWarehouses) : [];
  warehouses.value = resultWarehouses.map(item => ({
    ...item,
    date: dayjs(item.date)
  }));

  filteredDataFinish.value = [];
}

watch(transformedCompanySelected, () => {
  initValues();
});

// СПИСОК СКЛАДОВ, КОТОРЫЙ ПРИШЕЛ ИЗ API
const warehousesOptions = ref([]);

// СПИСОК СКЛАДОВ, КОТОРЫЕ ВЫБРАЛ ПОЛЬЗОВАТЕЛЬ
const warehousesSelected = ref([]);

const warehouses = ref([]);

watch(warehouses, (newValue) => {
  localStorage.setItem(fieldCompanies("warehouses_limit"), JSON.stringify(newValue));
}, { deep: true });

// ПОЛУЧАЕМ ВЕСЬ СПИСОК СКЛАДОВ (БЕЗ ПАРАМЕТРОВ)
watch(transformedCompanySelected, (newValue) => {
  axios.get("https://supplies-api.wildberries.ru/api/v1/warehouses", {
    headers: {
      Authorization: `${newValue.apiToken}`
    }
  })
    .then(response => {
      console.log(response.data);

      const transformData = response.data.map((warehouse) => ({
        ID: warehouse.ID,
        name: warehouse.name,
      }));

      warehousesOptions.value = transformData.map((warehouse) => ({
        label: warehouse.name,
        value: JSON.stringify(warehouse),
      }));
    })
    .catch(error => {
      console.error("Ошибка при получении складов\n", error.message);
    });
}, { immediate: true });

watch(warehousesSelected, (newValue) => {
  // Сохранить выбранные склады
  localStorage.setItem(fieldCompanies("warehouses_selected_limit"), JSON.stringify(newValue));

  const newArray = newValue.map((warehouse) => JSON.parse(warehouse));

  // Создаем временный массив, чтобы сохранять итоговый результат
  const updatedWarehouses = [];

  newArray.forEach((parsedWarehouse) => {
    // Проверяем, существует ли объект с таким же ID в warehouses.value
    const existingWarehouse = warehouses.value.find(
      (item) => item.ID === parsedWarehouse.ID
    );

    if (existingWarehouse) {
      // Если объект уже существует, добавляем его без изменений
      updatedWarehouses.push(existingWarehouse);
    } else {
      // Если объект новый, добавляем его с новыми значениями
      updatedWarehouses.push({
        ...parsedWarehouse,
        coefficientFrom: 0,
        coefficientTo: 2,
        date: dayjs().startOf('day'),
        deliveryTypeBox: true,
        deliveryTypeMonopallet: false,
        searchWithinWeek: false
      });
    }
  });

  // Обновляем warehouses.value, чтобы включить только актуальные данные
  warehouses.value = updatedWarehouses;
});

function updateWarehouse(warehouse) {
  const index = warehouses.value.findIndex(item => item.ID === warehouse.ID);
  if (index !== -1) {
    warehouses.value[index] = { ...warehouses.value[index], ...warehouse };
  }
}

const filteredDataFinish = ref([]);

// ПОЛУЧАЕМ СКЛАДЫ С КОЭФФИЦИЕНТАМИ
function coefficientsGet() {
  const idsArray = warehouses.value.map(warehouse => warehouse.ID);
  const paramsStr = idsArray.join(',');

  axios.get("https://supplies-api.wildberries.ru/api/v1/acceptance/coefficients", {
    params: {
      warehouseIDs: paramsStr
    },
    headers: {
      Authorization: `${transformedCompanySelected.value.apiToken}`
    }
  })
    .then(response => {
      const filteredData = response.data?.filter(resItem => {
        return warehouses.value.some(whItem => {
          const isWarehouse = resItem.warehouseID === whItem.ID;

          const resDate = dayjs(resItem.date).format("YYYY-MM-DD");
          const whDate = dayjs(whItem.date).format("YYYY-MM-DD");
          const isDate = whItem.searchWithinWeek
            ? resDate >= whDate && resDate <= dayjs(whItem.date).add(7, 'day').format("YYYY-MM-DD")
            : resDate >= whDate;

          const isWithinCoefficientRange = resItem.coefficient >= whItem.coefficientFrom && resItem.coefficient <= whItem.coefficientTo;

          const isBoxTypeCorrect = (whItem.deliveryTypeBox && resItem.boxTypeID === 2) || (whItem.deliveryTypeMonopallet && resItem.boxTypeID === 5);

          return isWarehouse && isDate && isWithinCoefficientRange && isBoxTypeCorrect;
        });
      });

      filteredDataFinish.value = filteredData ? filteredData : [];
    })
    .catch(error => {
      console.error("Ошибка при получении коэффициентов\n", error.message);
    });
}

watch(filteredDataFinish, (newArray, oldArray) => {
  // Поиск отсутствующего элемента
  for (let i = 0; i < oldArray.length; i++) {
    if (!newArray.find(item => JSON.stringify(item) === JSON.stringify(oldArray[i]))) {

      sendMessageToTelegram(oldArray[i], false);
    }
  }

  // Проверка и вывод новых изменений
  for (let i = 0; i < newArray.length; i++) {
    if (JSON.stringify(newArray[i]) !== JSON.stringify(oldArray[i])) {

      sendMessageToTelegram(newArray[i], true);
    }
  }
});

const getCurrentDateTime = () => {
  return new Date().toLocaleTimeString(navigator.language);
};

async function sendMessageToTelegram(options, status) {
  const { date, coefficient, warehouseName, boxTypeName } = options;

  const url = `https://api.telegram.org/bot${transformedCompanySelected.value.telegramToken}/sendMessage`;

  const acceptance = coefficient === 0 ? "Бесплатно" : `x${ coefficient }`;

  // const formattedMessage =
  //   `${ status ? '*Лимит найден 🟢*' : '*Лимит удалён 🔴*' }\n` +
  //   `${ getCurrentDateTime() }\n` +
  //   `\n` +
  //   `*Дата:* ${ dayjs(date).format('DD.MM.YYYY') }\n` +
  //   `*Поставка:* ${ warehouseName }, ${ boxTypeName }\n` +
  //   `*Приёмка:* ${ acceptance }`;

  const formattedMessage =
    `${ status ? '🟢' : '🔴' } ${ warehouseName }\n` +
    `${ dayjs(date).format('DD.MM.YYYY') }\n` +
    `${ acceptance }\n` +
    `\n` +
    `${ getCurrentDateTime() }`;

  try {
    await axios.post(url, {
      chat_id: transformedCompanySelected.value.chatId,
      text: formattedMessage.trim(),
      parse_mode: 'Markdown'
    });
  } catch (error) {
    console.error('Ошибка отправки сообщения\n', error.message);
  }
}
</script>

<template>
  <a-config-provider :locate="ruRu">
    <a-form ref="form" layout="vertical">
      <a-row :gutter="24">
        <a-col :span="12">
          <a-form-item label="Компания" name="companySelected">
            <a-select
              v-model:value="companySelected"
              :options="companyOptions"
              placeholder="Выберите из списка"
              show-search
            />
          </a-form-item>
        </a-col>
      </a-row>

      <a-row :gutter="24">
        <a-col :span="12">
          <a-form-item label="Склад" name="warehousesSelected">
            <a-select
              v-model:value="warehousesSelected"
              :options="warehousesOptions"
              :max-tag-count="10"
              mode="multiple"
              placeholder="Выберите из списка"
              show-search
              allow-clear
            />
          </a-form-item>
        </a-col>
      </a-row>
    </a-form>

    <div class="campaign-list">
      <CardWarehouse
        v-for="warehouse in warehouses"
        :key="warehouse.ID"
        :warehouse="warehouse"
        @updateWarehouse="updateWarehouse($event)"
      />
    </div>
  </a-config-provider>
</template>

<style scoped>
.campaign-list {
  display: flex;
  flex-direction: column;
  row-gap: 20px;
}
</style>
