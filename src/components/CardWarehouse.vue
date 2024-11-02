<script setup>
import { ref, watch } from "vue";
import {
  Col as ACol,
  Form as AForm,
  FormItem as AFormItem,
  Row as ARow,
  Checkbox as ACheckbox,
  Input as AInput,
  Card as ACard,
  DatePicker as ADatePicker,
  ConfigProvider as AConfigProvider,
} from "ant-design-vue";

import ruRu from "ant-design-vue/es/locale/ru_RU";
import dayjs from "dayjs";
import ru from "dayjs/locale/ru";
import utc from "dayjs/plugin/utc";

dayjs.locale(ru);
dayjs.extend(utc);

const props = defineProps({
  warehouse: {
    type: Object,
    required: true,
  }
});

const emit = defineEmits([
  "updateWarehouse"
]);

// Локальная копия warehouse с возможностью редактирования
const warehouseLocal = ref({ ...props.warehouse });

// Обновляем warehouseLocal только при изменении props.warehouse
watch(
  () => props.warehouse,
  (newWarehouse) => {
    // Обновляем только если данные реально поменялись, чтобы избежать зацикливания
    if (JSON.stringify(newWarehouse) !== JSON.stringify(warehouseLocal.value)) {
      warehouseLocal.value = { ...newWarehouse };
    }
  },
  { deep: true }
);

// Следим за изменениями в warehouseLocal и эмитим обновленное значение
watch(warehouseLocal, (newValue) => {
  emit("updateWarehouse", newValue);
}, { deep: true });

// ФОРМАТ ОТОБРАЖЕНИЯ ДАТЫ
const dateFormat = "DD.MM.YYYY";
</script>

<template>
  <a-config-provider :locale="ruRu">
    <a-card class="card">
      <a-form ref="form" layout="vertical">
        <a-row :gutter="24">
          <a-col :span="5">
            <div class="card__label">
              <span class="big-size">{{ warehouseLocal.name }}</span>
              <span class="grey">ID {{ warehouseLocal.ID }}</span>
            </div>
          </a-col>

          <a-col :span="4">
            <a-form-item
              label="Коэффициент до"
              name="coefficientTo"
            >
              <a-input
                v-model:value.number="warehouseLocal.coefficientTo"
                placeholder="Значение до"
              >
              </a-input>
            </a-form-item>
          </a-col>

          <a-col :span="4">
            <a-form-item label="Дата" name="warehouseDate"
            >
              <a-date-picker
                v-model:value="warehouseLocal.date"
                :format="dateFormat"
              />
            </a-form-item>
          </a-col>

          <a-col :span="3">
            <a-form-item label="Тип поставки" name="deliveryTypeBox">
              <a-checkbox v-model:checked="warehouseLocal.deliveryTypeBox">
                Короб
              </a-checkbox>
            </a-form-item>
          </a-col>

          <a-col :span="3">
            <a-form-item label="Тип поставки" name="deliveryTypeMonopallet">
              <a-checkbox v-model:checked="warehouseLocal.deliveryTypeMonopallet">
                Монопаллет
              </a-checkbox>
            </a-form-item>
          </a-col>

<!--          <a-col :span="10">-->
<!--            <div class="card__label">-->
<!--              <span class="big-size">{{ warehouseLocal.name }}</span>-->
<!--              <span class="grey">ID {{ warehouseLocal.ID }}</span>-->
<!--            </div>-->
<!--          </a-col>-->
        </a-row>
      </a-form>
    </a-card>
  </a-config-provider>
</template>

<style scoped>
.card {
  background-color: rgba(0, 0, 0, 0.01);
}

.card__label {
  display: flex;
  flex-direction: column;
}

.big-size {
  color: rgba(0, 0, 0, 0.88);
  font-weight: 600;
  font-size: 16px;
}

.grey {
  color: rgba(0, 0, 0, 0.5);
}
</style>
