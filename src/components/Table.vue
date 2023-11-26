<script>
import myJSON from "../assets/data.json";

export default {
  data() {
    return {
      jsonData: myJSON,
      sortOrder: "asc",
      filterStatus: "",
    };
  },

  computed: {
    filteredData() {
      console.log(this.filterStatus);
      if (this.filterStatus === "") {
        return this.jsonData;
      } else {
        return this.jsonData.filter((item) => {
          const statusDescription = this.printDeadline(
            item.dates[0].start_date,
            item.dates[0].end_date
          ).statusDescription;
          console.log(statusDescription.includes(this.filterStatus));
          return statusDescription.includes(this.filterStatus);
        });
      }
    },
  },

  methods: {
    /**
     * Сортирует данные таблицы по столбцу "Статус".
     *
     * Если в одной ячейке несколько значений, то сортировка происходит по первому значению.
     * Поэтому нужно уделять дополнительное внимание тому, какие значения есть в подобных ячейке.
     *
     * Сортировка происходит по следующим правилам (для сортировки по возрастанию):
     * 1. Сначала идут заказы, начинаются через X дней.
     * 2. Затем идут заказы, которые начинаются завтра.
     * 3. Затем идут заказы, которые начинаются сегодня.
     * 4. Затем идут заказы, которые заканчиваются через X дней.
     * 5. Затем идут заказы, которые заканчиваются завтра.
     * 6. Затем идут заказы, которые заканчиваются сегодня.
     * 7. Затем идут заказы, которые закончились вчера.
     * 8. Затем идут заказы, которые закончились X дней назад.
     *
     */
    sortData() {
      this.filteredData.sort((a, b) => {
        const statusA = this.printDeadline(
          a.dates[0].start_date,
          a.dates[0].end_date
        );
        const statusB = this.printDeadline(
          b.dates[0].start_date,
          b.dates[0].end_date
        );

        if (this.sortOrder === "asc") {
          if (statusA.diffDaysFromStart < 0 && statusB.diffDaysFromStart < 0) {
            return statusB.diffDaysFromStart - statusA.diffDaysFromStart;
          } else if (
            statusA.diffDaysFromStart === 0 &&
            statusB.diffDaysFromStart === 0
          ) {
            return statusA.diffDaysToEnd - statusB.diffDaysToEnd;
          } else if (
            statusA.diffDaysFromStart > 0 &&
            statusB.diffDaysFromStart > 0
          ) {
            if (statusA.diffDaysToEnd === 0 && statusB.diffDaysToEnd === 0) {
              return statusA.diffDaysFromStart - statusB.diffDaysFromStart;
            } else if (
              statusA.diffDaysToEnd === 1 &&
              statusB.diffDaysToEnd === 1
            ) {
              return statusA.diffDaysFromStart - statusB.diffDaysFromStart;
            } else if (
              statusA.diffDaysToEnd === 0 &&
              statusB.diffDaysToEnd === 1
            ) {
              return -1;
            } else if (
              statusA.diffDaysToEnd === 1 &&
              statusB.diffDaysToEnd === 0
            ) {
              return 1;
            } else {
              return statusA.diffDaysToEnd - statusB.diffDaysToEnd;
            }
          } else if (
            statusA.diffDaysFromStart > 0 &&
            statusB.diffDaysFromStart === 0
          ) {
            return -1;
          } else if (
            statusA.diffDaysFromStart === 0 &&
            statusB.diffDaysFromStart > 0
          ) {
            return 1;
          } else if (
            statusA.diffDaysFromStart > 0 &&
            statusB.diffDaysFromStart < 0
          ) {
            return -1;
          } else if (
            statusA.diffDaysFromStart < 0 &&
            statusB.diffDaysFromStart > 0
          ) {
            return 1;
          }
        } else if (this.sortOrder === "desc") {
          if (statusA.diffDaysFromStart < 0 && statusB.diffDaysFromStart < 0) {
            return statusA.diffDaysFromStart - statusB.diffDaysFromStart;
          } else if (
            statusA.diffDaysFromStart === 0 &&
            statusB.diffDaysFromStart === 0
          ) {
            return statusB.diffDaysToEnd - statusA.diffDaysToEnd;
          } else if (
            statusA.diffDaysFromStart > 0 &&
            statusB.diffDaysFromStart > 0
          ) {
            if (statusA.diffDaysToEnd === 0 && statusB.diffDaysToEnd === 0) {
              return statusB.diffDaysFromStart - statusA.diffDaysFromStart;
            } else if (
              statusA.diffDaysToEnd === 1 &&
              statusB.diffDaysToEnd === 1
            ) {
              return statusB.diffDaysFromStart - statusA.diffDaysFromStart;
            } else if (
              statusA.diffDaysToEnd === 0 &&
              statusB.diffDaysToEnd === 1
            ) {
              return 1;
            } else if (
              statusA.diffDaysToEnd === 1 &&
              statusB.diffDaysToEnd === 0
            ) {
              return -1;
            } else {
              return statusB.diffDaysToEnd - statusA.diffDaysToEnd;
            }
          } else if (
            statusA.diffDaysFromStart > 0 &&
            statusB.diffDaysFromStart === 0
          ) {
            return 1;
          } else if (
            statusA.diffDaysFromStart === 0 &&
            statusB.diffDaysFromStart > 0
          ) {
            return -1;
          } else if (
            statusA.diffDaysFromStart > 0 &&
            statusB.diffDaysFromStart < 0
          ) {
            return 1;
          } else if (
            statusA.diffDaysFromStart < 0 &&
            statusB.diffDaysFromStart > 0
          ) {
            return -1;
          }
        }

        return 0;
      });

      this.sortOrder = this.sortOrder === "asc" ? "desc" : "asc";
    },

    /**
     * Получает две даты и возвращает строку с описанием статуса заказа.
     *
     * Предусмотрены следующие статусы:
     * 1. Заказ заканчивается сегодня.
     * 2. Заказ заканчивается завтра.
     * 3. Заказ заканчивается через X дней.
     * 4. Заказ закончился вчера.
     * 5. Заказ закончился X дней назад.
     * 6. Заказ начинается сегодня.
     * 7. Заказ начинается завтра.
     * 8. Заказ начинается через X дней.
     * @param {*} soonDateStart Дата начала заказа
     * @param {*} soonDateEnd Дата конца заказа
     */
    printDeadline(soonDateStart, soonDateEnd) {
      const startDate = new Date(soonDateStart + "T00:00:00Z");
      const endDate = new Date(soonDateEnd + "T00:00:00Z");

      const today = new Date();
      today.setUTCHours(0, 0, 0, 0);

      const diffTimeToEnd = endDate - today;
      const diffDaysToEnd = Math.floor(diffTimeToEnd / (1000 * 60 * 60 * 24));

      const diffTimeFromStart = today - startDate;
      const diffDaysFromStart = Math.floor(
        diffTimeFromStart / (1000 * 60 * 60 * 24)
      );

      // Статус заказа
      let statusDescription = "";
      if (diffDaysFromStart === 0) {
        statusDescription = "Заказ начинается сегодня";
      } else if (diffDaysToEnd === 0) {
        statusDescription = "Заказ заканчивается сегодня";
      } else if (diffDaysFromStart === 1) {
        statusDescription = "Заказ начинается завтра";
      } else if (diffDaysFromStart < 0) {
        statusDescription = `Заказ начинается через ${-diffDaysFromStart} дней`;
      } else if (diffDaysToEnd === 1) {
        statusDescription = "Заказ заканчивается завтра";
      } else if (diffDaysToEnd > 1) {
        statusDescription = `Заказ заканчивается через ${diffDaysToEnd} дней`;
      } else if (diffDaysToEnd === -1) {
        statusDescription = "Заказ закончился вчера";
      } else if (diffDaysToEnd < -1) {
        statusDescription = `Заказ закончился ${Math.abs(
          diffDaysToEnd
        )} дней назад`;
      }

      return { statusDescription, diffDaysToEnd, diffDaysFromStart };
    },
  },

  /**
   * ФУНКЦИЯ НЕ РАБОТАЕТ!
   *
   * Применяет фильтр к данным таблицы.
   */
  applyFilter() {
    if (this.filterStatus === "") {
      this.filteredData = this.jsonData;
    } else if (this.filterStatus === "заканчивается через") {
      this.filteredData = this.jsonData.filter((item) => {
        const deadline = this.printDeadline(
          item.dates[0].start_date,
          item.dates[0].end_date
        );
        return deadline.diffDaysToEnd > 0;
      });
    } else if (this.filterStatus === "заканчивается завтра") {
      this.filteredData = this.jsonData.filter((item) => {
        const deadline = this.printDeadline(
          item.dates[0].start_date,
          item.dates[0].end_date
        );
        return deadline.diffDaysToEnd === 1;
      });
    } else if (this.filterStatus === "заканчивается сегодня") {
      this.filteredData = this.jsonData.filter((item) => {
        const deadline = this.printDeadline(
          item.dates[0].start_date,
          item.dates[0].end_date
        );
        return deadline.diffDaysToEnd === 0;
      });
    } else if (this.filterStatus === "закончился вчера") {
      this.filteredData = this.jsonData.filter((item) => {
        const deadline = this.printDeadline(
          item.dates[0].start_date,
          item.dates[0].end_date
        );
        return deadline.diffDaysToEnd === -1;
      });
    } else if (this.filterStatus === "закончился") {
      this.filteredData = this.jsonData.filter((item) => {
        const deadline = this.printDeadline(
          item.dates[0].start_date,
          item.dates[0].end_date
        );
        return deadline.diffDaysToEnd < -1;
      });
    }
  },
};
</script>

<template>
  <!-- ПОЛЕ С ФИЛЬТРОМ -->
  <div id="filter" style="position: relative">
    <label for="statusFilter">Фильтр по статусу:</label>
    <select v-model="filterStatus" id="statusFilter" @change="applyFilter">
      <option value="">Все</option>
      <option value="заканчивается через">
        Заказ заканчивается через X дней
      </option>
      <option value="заканчивается завтра">Заказ заканчивается завтра</option>
      <option value="заканчивается сегодня">Заказ заканчивается сегодня</option>
      <option value="закончился вчера">Заказ закончился вчера</option>
      <option value="закончился">Заказ закончился X дней назад</option>
    </select>
  </div>

  <!-- ПОЛЕ С ТАБЛИЦЕЙ -->
  <div id="field">
    <table>
      <thead>
        <tr>
          <th>Номер заказа</th>
          <th>Клиент</th>
          <th>Диеты</th>
          <th>Тарифы</th>
          <th>Адрес</th>
          <th>Телефон</th>
          <th>Даты</th>
          <th>Скидка</th>
          <th>Статус оплаты</th>
          <th>Комментарий курьера</th>
          <th>Внутренний комментарий</th>
          <th @click="sortData">Статус</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in jsonData" :key="item.o_id">
          <!-- ID КЛИЕНТА -->
          <td>{{ item.o_id }}</td>

          <!-- ФИО КЛИЕНТА -->
          <td>{{ item.client_name }}</td>

          <!-- ДИЕТЫ -->
          <td>
            <span v-for="(diet, index) in item.diets" :key="index">
              {{ diet }}
              <div id="separator" v-if="index !== item.diets.length - 1"></div>
            </span>
          </td>

          <!-- ТАРИФЫ -->
          <td>
            <span v-for="(tariff, index) in item.tariff" :key="index">
              {{ tariff }}
              <div id="separator" v-if="index !== item.tariff.length - 1"></div>
            </span>
          </td>

          <!-- АДРЕС -->
          <td>{{ item.address }}</td>

          <!-- НОМЕР ТЕЛЕФОНА -->
          <td>{{ item.phone }}</td>

          <!-- ДАТЫ -->
          <td style="width: 16rem">
            <span v-for="(date, index) in item.dates" :key="index">
              От {{ date.start_date }} до {{ date.end_date }}
              <div id="separator" v-if="index !== item.dates.length - 1"></div>
            </span>
          </td>

          <!-- СКИДКА -->
          <td>{{ item.discount }}</td>

          <!-- СТАТУС ОПЛАТЫ,  PAYMENT STATUS, COST + DEPT -->
          <td
            v-if="item.pay_status == 'Оплачен'"
            style="
              background-color: limegreen;
              font-weight: bold;
              flex-direction: column;
            "
          >
            <span> Стоимость: {{ item.order_sum }} </span>
            <div id="separator"></div>
            <span>{{ item.pay_status }}</span>
            <div id="separator"></div>
            <span>
              Долг:
              {{
                item.order_sum - item.order_payed >= 0
                  ? item.order_sum - item.order_payed
                  : 0
              }}
            </span>
          </td>
          <td
            v-if="item.pay_status == 'Неоплачен ч.'"
            style="
              background-color: rgb(255, 80, 80);
              font-weight: bold;
              flex-direction: column;
            "
          >
            <span> Стоимость: {{ item.order_sum }} </span>
            <div id="separator"></div>
            <span>{{ item.pay_status }}</span>
            <div id="separator"></div>
            <span> Долг: {{ item.order_sum - item.order_payed }} </span>
          </td>

          <!-- COURIER COMMENT -->
          <td
            v-if="item.courier_comment"
            style="background-color: rgb(253, 253, 105); font-weight: bold"
          >
            {{ item.courier_comment }}
          </td>
          <td v-else>{{ item.courier_comment }}</td>

          <!-- INNER COMMENT -->
          <td
            v-if="item.inner_comment"
            style="background-color: rgb(253, 253, 105); font-weight: bold"
          >
            {{ item.inner_comment }}
          </td>
          <td v-else>{{ item.inner_comment }}</td>

          <!-- DEADLINE -->
          <td>
            <span v-for="(date, index) in item.dates" :key="index">
              {{
                printDeadline(date.start_date, date.end_date).statusDescription
              }}

              <div id="separator" v-if="index !== item.dates.length - 1"></div>
            </span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped lang="scss">
$UI-orange: rgb(255, 166, 0);
$UI-cyan: rgb(0, 162, 255);
$UI-green: rgb(127, 238, 83);
$UI-white: rgb(255, 255, 255);
$UI-light-gray: rgb(209, 209, 209);
$UI-blue-name: rgb(0, 162, 255);

$UI-deadline-light: rgb(221, 160, 221);
$UI-deadline-dark: rgb(211, 116, 211);

$row-hover: #8f8f8f;

$padding-main: 1rem;

#filter {
  margin: 1rem;

  label {
    margin-right: 0.5rem;
  }

  select {
    padding: 0.25rem;
  }
}

#separator {
  margin: 0.5rem 0;
  border-bottom: 1px solid #ddd;
  width: 100%;
  border-bottom: 3px;
  border-color: black;
  border-bottom-style: dashed;
}

#field {
  margin: 0;
  width: max-content;
  display: flex;
  justify-content: center;
  align-items: center;
}

table {
  color: black;

  th,
  td {
    text-align: center;
  }

  tr:nth-child(even) {
    background-color: $UI-light-gray;
  }

  th {
    background-color: $UI-orange;
    color: rgb(0, 0, 0);
    width: fit-content;
    padding: $padding-main;
  }

  tr {
    background-color: $UI-white;

    &:hover,
    &:focus {
      background-color: $row-hover;
    }
  }

  td {
    border-bottom: 1px solid #ddd;
    border-left: 1px solid #ddd;
    width: 10rem;
    padding: 0;
    margin: 0;

    span {
      display: flex;
      flex-direction: column;
      width: 100%;
    }

    &:first-child {
      border-left: 1px solid #81c965;
      background-color: $UI-green;
      color: rgb(0, 0, 0);
      font-weight: bold;
      width: 1rem;
    }

    &:nth-child(2) {
      color: $UI-blue-name;
      font-weight: bolder;
      padding: 5px;
    }

    &:nth-last-child(1) {
      background-color: $UI-deadline-light;
      font-weight: bold;
    }

    &:nth-child(7) {
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }

    &:nth-child(9) {
      width: fit-content;
    }

    &:nth-child(10),
    &:nth-child(11) {
      padding: 5px;
    }

    &:nth-child(12) {
      width: 20rem;
    }
  }

  tr:hover {
    background-color: $row-hover;

    & td {
      color: black;
    }

    & td:first-child {
      background-color: green;
    }

    & td:last-child {
      background-color: $UI-deadline-dark;
    }
  }

  th:hover {
    background-color: rgb(255, 102, 0);
  }
}
</style>
