<template>
  <div
    class="col summary-section"
    :class="{ 'is-collapsed': !isNotesExpanded }"
  >
    <!-- <button class="toggle-notes" @click="isNotesExpanded = !isNotesExpanded">
        {{ isNotesExpanded ? "✕ Ukryj" : "☰ Pokaż zamówienia" }}
      </button> -->
    <div>
      <h2 v-if="isFullOrderVisible">Propozycja smaków</h2>
      <transition name="note-fade" v-if="isFullOrderVisible">
        <div
          v-if="isNotesExpanded && summary && summary.length > 0"
          class="order-list"
        >
          <h2 style="border: none">Zaznacz kto płaci</h2>
          <div v-for="(item, index) in summary" :key="index" class="order-card">
            <template v-if="item">
              <div class="order-header">
                <div class="user-info">
                  <input
                    v-if="isCurrentUserManager"
                    type="radio"
                    v-model="payingPerson"
                    :value="item.name"
                  />
                  <span v-if="isCurrentUserManager">zaznacz jeśli płaci</span>
                  <span class="customer-name">{{ item?.name }}</span>
                  <div class="counter-btns">
                    <button @click="updateSlice(item, -1)" title="Odejmij">
                      -
                    </button>
                    <button @click="updateSlice(item, 1)" title="Dodaj">
                      +
                    </button>
                    <button @click="DelateUser(item.name)">❌</button>
                  </div>
                </div>

                <div class="price-info">
                  <p class="slices-count">
                    zje <strong>{{ item.slice }}</strong>
                    {{
                      getPlural(item.slice, ["kawałek", "kawałki", "kawałków"])
                    }}
                  </p>
                  <span class="restaurant-tag">{{
                    item?.selectedRestaurant || "Nieznana"
                  }}</span>
                </div>
              </div>
              <div class="order-body">
                <ul class="pizza-list">
                  <li v-for="pId in item?.pizzas || []" :key="pId">
                    🍕
                    {{
                      getPizzaNameFromMenu(
                        pId,
                        item?.selectedRestaurant,
                        // item.price,
                      )
                    }}
                  </li>
                </ul>
              </div>
            </template>
          </div>
          <h2 v-if="isCurrentUserManager">Ostateczny wybór smaków</h2>
          <div
            v-for="user in payingUsers"
            :key="user.id"
            class="paying-user-badge"
          >
            płaci {{ user.name }}: <span class="phone">{{ user.phone }}</span>
          </div>
          <div v-if="isCurrentUserManager"></div>
          <div v-for="(pizzas, restaurant) in SummaryOrders" :key="restaurant">
            <h3>{{ restaurant }}</h3>
            <!-- {{ selectPizzas }} -->
            <ul v-if="isCurrentUserManager">
              <li v-for="(count, name) in pizzas" :key="name">
                <input type="checkbox" :value="name" v-model="checkedPizza" />
                {{ name }}: <span v-if="count < 2">{{ count }}</span>
                <select v-if="count > 1" v-model.number="selectPizzas[name]">
                  <option v-for="n in count" :key="n" :value="n">
                    {{ n }}
                  </option>
                </select>
                szt.
              </li>
            </ul>
          </div>
          <strong v-if="isCurrentUserManager" class="MenagerView">
            Łączna liczba kawałków {{ SummarySlices }} Łączna cena
            <span>{{ TotalMoney }}</span>
            <div v-if="payingPerson">
              <div class="payment-method-selection">
                <label>
                  <input type="radio" value="blik" v-model="paymentMethod" />
                  BLIK (podaj tel)
                </label>
                <label>
                  <input type="radio" value="gotowka" v-model="paymentMethod" />
                  Gotówka
                </label>
              </div>

              <div v-if="paymentMethod === 'blik'" class="paying-info">
                <h2 class="phone-number-paying-person">Podaj numer telefonu</h2>
                <input type="tel" v-model="tel" placeholder="000-000-000" />
                <p>Numer: {{ tel }}</p>
              </div>

              <div v-else class="paying-info">
                <p>
                  Wybrano płatność <strong>gotówką</strong> – numer telefonu nie
                  jest wymagany.
                </p>
              </div>
            </div>
            <button @click="approveFlavors">Zatwierdź smaki</button>
          </strong>
        </div>
      </transition>
    </div>

    <div class="orders-list">
      <h3>Zamówione pizze dzisiaj:</h3>
      <div
        v-if="orderedPizzas && orderedPizzas.find((item) => item && item.payer)"
      >
        <div
          v-for="order in [orderedPizzas.find((item) => item && item.payer)]"
          :key="order.payer.phone"
        >
          <ul class="ordered-pizzas-summary">
            <li
              v-for="(count, pizzaName) in order.selectedPizzas"
              :key="pizzaName"
            >
              <span v-if="pizzaName !== 'Ładowanie...'">
                🍕 <strong>{{ pizzaName }} x {{ count }}</strong>
              </span>
            </li>
          </ul>

          <div class="final-order-card">
            <p class="payer-info">
              💰 Płaci: <strong>{{ order.payer.name }}</strong>
            </p>
            <p class="phone-info">
              📞 Tel:
              <span v-if="order.payer.phone === 'Gotówka'"
                ><strong>Gotówka</strong></span
              >
              <a v-else :href="'tel:' + order.payer.phone" class="phone-link">
                {{ order.payer.phone }}
              </a>
            </p>
            <p class="total-price-display">
              Razem:
              <strong>{{ order.payer.totalToPayAtRestaurant }} zł</strong>
            </p>
          </div>

          <h4>💰 Kto ile oddaje:</h4>
          <div
            v-for="person in order.participants"
            :key="person.name"
            class="participant-row"
          >
            <span class="person-details">
              👤 {{ person.name }} ({{ person.slices }} szt.)
            </span>

            <strong
              v-if="person.name !== order.payer.name"
              class="person-amount"
            >
              {{ person.toPay }} zł
            </strong>

            <strong v-else> Ta osoba płaci </strong>
          </div>
        </div>
      </div>

      <div v-else class="empty-state">
        <p>nie ma zamówień LOL</p>
      </div>

      <hr class="divider" />

      <div class="admin-buttons">
        <button class="btn-clear" @click="clearApi">Wyczyść zamówienie</button>
      </div>
      <div class="history-section">
        <h3>📜 Historia zamówień</h3>

        <select
          v-model="selectedFileName"
          @change="loadOrderFromFile"
          class="date-input"
        >
          <option value="" disabled>Wybierz plik zamówienia</option>
          <option v-for="file in availableFiles" :key="file" :value="file">
            {{ file.replace(".json", "").replace(/_/g, ".") }}
          </option>
        </select>

        <div class="history-list">
          <div
            v-for="(order, index) in PersonsPaied"
            :key="index"
            class="history-item"
          >
            <div v-if="order && order.payer">
              <p>💰 <strong>Płacił:</strong> {{ order.payer.name }}</p>

              <div v-if="order.participants">
                <h4>💰 Lista rozliczeń:</h4>
                <ul>
                  <li v-for="p in order.participants" :key="p.name">
                    {{ p.name }} — <strong>{{ p.toPay }} zł</strong>

                    <input
                      type="checkbox"
                      v-model="p.paied"
                      @change="handleStatusChange"
                    />
                    <span :style="{ color: p.paied ? 'green' : 'red' }">
                      {{ p.paied ? "✅ Zapłacone" : "⏳ Czeka" }}
                    </span>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Cookies from "js-cookie";

export default {
  name: "orderSummary",
  data() {
    return {
      selectedRestaurant: null,
      allMenus: {},
      pizzas: [],
      summary: null,
      isNotesExpanded: true,
      checkedPizza: [],
      payingPerson: null,
      selectPizzas: {},
      FinalOrderSummary: {},
      orderedPizzas: [],
      Persons: [],
      FullOrderFlag: true,
      IsPaied: false,
      tel: "",
      paymentMethod: "blik",
      currentStep: 0,
      availableFiles: [], // Lista plików z getOrdersList
      selectedFileName: "", // Wybrany plik z selecta
      PersonsPaied: [],
      steps: [{ valid: false }, { valid: false }, { valid: false }],
    };
  },
  watch: {
    summary: {
      handler(newVal) {
        if (!newVal) return;
        newVal.forEach((item) => {
          if (item?.pizzas && item.selectedRestaurant) {
            item.pizzas.forEach((pId) => {
              const name = this.getPizzaNameFromMenu(
                pId,
                item.selectedRestaurant,
              );
              if (
                name !== "Ładowanie..." &&
                this.selectPizzas[name] === undefined
              ) {
                this.selectPizzas[name] = 1;
              }
            });
          }
        });
      },
      deep: true,
    },
    user: {
      handler(newOrders) {
        this.steps[0].valid = true;
        this.currentStep = 2;
        if (!newOrders) return;

        Object.values(newOrders).forEach((pizzasList) => {
          Object.keys(pizzasList).forEach((pizzaName) => {
            if (this.selectPizzas[pizzaName] === undefined) {
              this.selectPizzas[pizzaName] = 1;
            }
          });
        });
      },
      deep: true,
    },
    selectedRestaurant: {
      handler() {
        if (this.selectedRestaurant) {
          this.steps[1].valid = true;
          this.currentStep = 3;
          this.fetchMenu(this.selectedRestaurant);
        }
      },
    },
    pizzas: {
      handler() {
        this.steps[2].valid = this.pizzas.length > 0;
      },
      deep: true,
    },
  },
  methods: {
    getPlural(n, forms) {
      const v = Math.abs(parseInt(n) || 0);
      if (v === 1) return forms[0];
      const lastTwo = v % 100;
      if (lastTwo > 10 && lastTwo < 20) return forms[2];
      const lastDigit = v % 10;
      if (lastDigit >= 2 && lastDigit <= 4) return forms[1];
      return forms[2];
    },
    async fetchrestaurants() {
      try {
        const response = await fetch(
          "https://webwizards.home.pl/jacek/pizza/api/?method=getLocals",
        );
        this.restaurants = await response.json();
      } catch (error) {
        console.error(error);
      }
    },
    async fetchMenuToCache(key) {
      if (!key || this.allMenus[key]) return;
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getMenu&pizzeria=${key}`,
        );
        const data = await response.json();
        this.allMenus[key] = data;
      } catch (error) {
        console.error(error);
      }
    },
    async DelateUser(name) {
      const savedName = Cookies.get("user_name");

      if (name !== savedName) {
        alert("Tylko siebie możesz usunąć XD.");
        return;
      }

      if (!confirm(`Na pewno chcesz usunąć użytkownika ${name}?`)) return;

      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=deleteOrderByUserName`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ name: name }),
          },
        );

        if (response.ok) {
          await this.fetchSummary();
          console.log(`usunięty ${name}`);
        }
      } catch (error) {
        console.error(error);
      }
    },

    async fetchSummary() {
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getTodayOrder`,
        );
        this.summary = await response.json();
      } catch (error) {
        console.error(error);
      }
    },
    async UpdataOrder(bodyContent) {
      if (!bodyContent || !bodyContent.name) return;
      try {
        await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=updateTodayOrder`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(bodyContent),
          },
        );
        this.fetchSummary();
      } catch (error) {
        console.error("Błąd", error);
      }
    },
    async clearApi() {
      try {
        const clearResponse = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=clearTodayOrder`,
        );
        if (clearResponse.ok) {
          this.fetchSummary();
          this.orderedPizzas = [];
          location.reload();
        }
      } catch (error) {
        console.error(error);
      }
    },
    async updateSlice(item, change) {
      if (!item || !item.name) return;

      const savedName = Cookies.get("user_name");

      if (item.name !== savedName) {
        alert("Tylko swoje możesz edytować XD.");
        return;
      }

      const newCount = (parseFloat(item.slice) || 0) + change;

      if (newCount > 8) {
        return;
      }

      if (newCount <= 0) {
        const confirmRemoval = confirm(
          `Czy na pewno rezygnujesz z swojej pizzy?`,
        );

        if (confirmRemoval) {
          item.slice = 0;
          await this.UpdataOrder(item);
          this.fetchSummary();
        }
        return;
      }

      item.slice = newCount;
      this.UpdataOrder(item);
    },
    async approveFlavors() {
      if (!this.SummarySlices.includes("Idealnie")) {
        alert(
          "Błąd w ilości kawałków lub pizz (Wszystko jest napisane w czarnym kwadracie).",
        );
        return;
      }

      const person = this.summary.find(
        (p) => p && p.name === this.payingPerson,
      );

      if (!this.payingPerson) {
        alert("Wybierz osobę płacącą!");
        return;
      }

      let finalPhoneValue = "";

      if (this.paymentMethod === "gotowka") {
        finalPhoneValue = "Gotówka";
      } else {
        const rawPhone = this.tel || (person ? person.phone : "");
        const cleanPhone = rawPhone
          ? String(rawPhone).replace(/\s|-/g, "")
          : "";
        const phoneRegex = /^[0-9]{9}$/;

        if (!phoneRegex.test(cleanPhone)) {
          alert(
            "Przy płatności BLIK numer telefonu musi składać się z 9 cyfr!",
          );
          return;
        }
        finalPhoneValue = cleanPhone;
      }

      const participantsList = this.summary
        .filter((item) => item && item.name && item.selectedRestaurant)
        .map((item) => ({
          name: item.name,
          slices: item.slice,
          toPay: this.calculatePrice(item),
          restaurant: item.selectedRestaurant,
          paied: item.paied || false,
        }));

      if (participantsList.length === 0) {
        alert("Brak poprawnych danych zamówienia!");
        return;
      }

      const pizzasToOrder = {};
      this.checkedPizza.forEach((name) => {
        pizzasToOrder[name] = this.selectPizzas[name] || 1;
      });

      try {
        // Czyścimy poprzednie zamówienie przed ustawieniem nowego
        await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=clearTodayOrder`,
        );

        this.FinalOrderSummary = {
          selectedPizzas: pizzasToOrder,
          paied: person ? person.paied : null,
          payer: {
            name: this.payingPerson,
            phone: finalPhoneValue,
            totalToPayAtRestaurant: this.TotalMoney,
          },
          participants: participantsList,
        };

        const setResponse = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=setTodayOrder`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(this.FinalOrderSummary),
          },
        );

        if (setResponse.ok) {
          this.fetchOrderedPizzas();
        }
      } catch (error) {
        console.error("Wystąpił błąd podczas finalizacji:", error);
        alert("Wystąpił błąd serwera przy zapisie zamówienia.");
      }
    },
    async fetchOrderedPizzas() {
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getTodayOrder`,
        );
        const data = await response.json();
        this.orderedPizzas = Array.isArray(data) ? data : [data];
      } catch (error) {
        console.error("Błąd pobierania:", error);
      }
    },

    async GetPiedPerson(data) {
      if (!data) return;

      const [year, month, day] = data.split("-");
      const formattedDate = `${day}_${month}_${year}`;

      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getSpecyfiedOrder&name=${formattedDate}.json`,
        );

        if (!response.ok) {
          this.Persons = [];
          return;
        }

        const result = await response.json();

        if (!result) {
          this.Persons = [];
          return;
        }

        // Filtrujemy tablicę, aby pozbyć się ewentualnych nulli z danych
        const rawData = Array.isArray(result) ? result : [result];
        this.Persons = rawData.filter(
          (item) => item !== null && typeof item === "object",
        );
      } catch (error) {
        console.error("Błąd:", error);
        this.Persons = [];
      }
    },
    async AddPhoneNumer(tel) {
      try {
        const response = await fetch(
          "https://webwizards.home.pl/jacek/pizza/api/?method=getTodayOrder",
        );
        const data = await response.json();
        if (JSON.stringify(data).includes("phone")) {
          alert("Ktoś już zaznaczył, że zapłaci nie wyrywaj się");
        } else {
          const person = this.summary.find(
            (p) => p && p.name === this.payingPerson,
          );
          if (person) {
            person.phone = tel;
            this.UpdataOrder(person);
          }
        }
      } catch (error) {
        console.error(error);
      }
    },

    async fetchFilesList() {
      try {
        const response = await fetch(
          "https://webwizards.home.pl/jacek/pizza/api/?method=getOrdersList",
        );
        const files = await response.json();
        // Odwracamy listę, żeby najnowsze daty były na górze
        this.availableFiles = Array.isArray(files) ? files.reverse() : [];
      } catch (error) {
        console.error("Błąd pobierania listy plików:", error);
      }
    },

    async loadOrderFromFile() {
      if (!this.selectedFileName) return;
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getSpecyfiedOrder&name=${this.selectedFileName}`,
        );
        const result = await response.json();

        // Konwersja na tablicę i filtrowanie nulli
        const rawData = Array.isArray(result) ? result : [result];
        this.PersonsPaied = rawData.filter(
          (item) => item !== null && typeof item === "object",
        );
      } catch (error) {
        console.error("Błąd ładowania pliku:", error);
        this.PersonsPaied = [];
      }
    },

    async handleStatusChange() {
      if (!this.selectedFileName || this.PersonsPaied.length === 0) {
        console.error("Brak pliku lub danych do zapisu");
        return;
      }

      const url = `https://webwizards.home.pl/jacek/pizza/api/?method=updateOrderByName&file=${this.selectedFileName}`;
      const payload = JSON.stringify(this.PersonsPaied);
      console.log(this.selectedFileName);
      
      try {
        const response = await fetch(url, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: payload,
        });

        if (response.ok) {
          console.log("Zapisano:", this.selectedFileName);

        } else {
          console.error("Serwer zwrócił błąd:", response.status);
        }
      } catch (error) {
        console.error("Błąd sieci:", error);
      }
    },

    getPizzaNameFromMenu(pizzaId, restaurantKey) {
      const targetMenu = this.allMenus[restaurantKey];
      if (targetMenu) {
        const pizza = targetMenu.find((p) => p.id == pizzaId);
        return pizza ? pizza.name : "Nieznana pizza";
      }
      this.fetchMenuToCache(restaurantKey);
      return "Ładowanie...";
    },
    getPizzaPrice(pizzaId, restaurantKey) {
      const targetMenu = this.allMenus[restaurantKey];
      if (targetMenu) {
        const pizza = targetMenu.find((p) => p.id == pizzaId);
        return pizza?.variants?.[1]?.price || 0;
      }
      return 0;
    },
    initPizzasSelect() {
      return "";
    },

    calculatePrice(item) {
      const totalCost = parseFloat(this.TotalMoney) || 0;
      const totalSlices = this.totalSlicesCount;

      if (totalCost === 0 || totalSlices === 0) return "0.00";

      const pricePerSlice = totalCost / totalSlices;
      let finalPrice = pricePerSlice * (parseInt(item.slice) || 0);

      if (Math.floor(finalPrice) === 21) {
        return "21.37 LOL XD";
      }

      return finalPrice.toFixed(2);
    },
    WhoPaied() {
      return this.summary ? this.summary.find((p) => p && p.phone) : null;
    },
  },
  computed: {
    SummaryOrders() {
      const totals = {};
      if (!this.summary || !Array.isArray(this.summary)) return totals;
      this.summary.forEach((element) => {
        if (!element) return;
        const restaurantKey = element.selectedRestaurant;
        if (!totals[restaurantKey]) totals[restaurantKey] = {};
        if (element.pizzas && Array.isArray(element.pizzas)) {
          element.pizzas.forEach((pId) => {
            const pName = this.getPizzaNameFromMenu(pId, restaurantKey);
            totals[restaurantKey][pName] =
              (totals[restaurantKey][pName] || 0) + 1;
          });
        }
      });
      return totals;
    },

    isPayer(name) {
      const currentPayer = this.payingPerson;
      const finalOrderPayer = this.orderedPizzas?.[0]?.payer?.name;

      return name === currentPayer || name === finalOrderPayer;
    },

    isCurrentUserManager() {
      const savedName = Cookies.get("user_name");
      if (!savedName || !this.summary) return false;
      const currentUser = this.summary.find((p) => p && p.name === savedName);
      return currentUser && currentUser.manager === true;
    },
    isFullOrderVisible() {
      if (!this.summary || !Array.isArray(this.summary)) return false;
      return this.summary.some((item) => item && item.name);
    },
    TotalMoney() {
      let total = 0;
      if (!this.checkedPizza || this.checkedPizza.length === 0) return "0.00";

      this.checkedPizza.forEach((pizzaName) => {
        const quantity = parseInt(this.selectPizzas[pizzaName]) || 1;
        let price = 0;

        for (const restaurant in this.allMenus) {
          const pizzaInMenu = this.allMenus[restaurant].find(
            (p) => p.name === pizzaName,
          );
          if (pizzaInMenu) {
            // Tu również stosujemy zasadę 21.37 dla pojedynczej pizzy
            let pPrice = parseFloat(pizzaInMenu.variants?.[1]?.price || 0);
            price = Math.floor(pPrice) === 21 ? 21.37 : pPrice;
            break;
          }
        }
        total += price * quantity;
      });

      // Dodatkowe sprawdzenie dla sumy końcowej całego zamówienia
      if (Math.floor(total) === 21) {
        return "21.37";
      }

      return total.toFixed(2);
    },
    SlicePrice() {
      return 0;
    },
    SummarySlices() {
      if (!this.summary || !Array.isArray(this.summary)) return "0";

      const KAWAŁKI_W_PIZZY = 8;
      const totalWantedSlices = this.summary.reduce((total, item) => {
        return total + (parseInt(item?.slice) || 0);
      }, 0);

      // Sprawdzamy, czy suma kawałków dzieli się przez 8 bez reszty
      const remainder = totalWantedSlices % KAWAŁKI_W_PIZZY;

      if (remainder !== 0) {
        const missing = KAWAŁKI_W_PIZZY - remainder;
        return `${totalWantedSlices} kawałków. Brakuje jeszcze ${missing} do pełnej pizzy!`;
      }

      // Jeśli mamy wielokrotność 8, sprawdzamy czy liczba wybranych pizz się zgadza
      const requiredPizzas = totalWantedSlices / KAWAŁKI_W_PIZZY;
      let pizzasSelected = 0;
      this.checkedPizza.forEach((pName) => {
        pizzasSelected += parseInt(this.selectPizzas[pName]) || 1;
      });

      const diffPizzas = requiredPizzas - pizzasSelected;

      if (diffPizzas > 0) {
        return `Masz ${totalWantedSlices} kawałków (pełne pizze). Do dodania ${diffPizzas} ${this.getPlural(
          diffPizzas,
          ["pizza", "pizze", "pizze"],
        )}.`;
      } else if (diffPizzas === 0) {
        return `${totalWantedSlices} kawałków. Idealnie! Pełne pizze (${pizzasSelected} szt).`;
      } else {
        return `Wybrałeś za dużo smaków względem liczby kawałków!`;
      }
    },
    totalSlicesCount() {
      if (!this.summary || !Array.isArray(this.summary)) return 0;
      return this.summary.reduce(
        (total, item) => total + (parseInt(item?.slice) || 0),
        0,
      );
    },
    payingUsers() {
      return this.summary ? this.summary.filter((u) => u?.phone) : [];
    },

    ShowArrear() {
      return 0;
    },
  },
  mounted() {
    this.fetchFilesList(); // <--- To dodaj
    this.fetchrestaurants();
    this.fetchSummary();
    this.fetchOrderedPizzas();

    this.polling = setInterval(() => {
      this.fetchSummary();
      this.fetchOrderedPizzas();
    }, 2000);
  },
  beforeUnmount() {
    clearInterval(this.polling);
  },
};
</script>
<style lang="scss" scoped>
/* GŁÓWNY KONTENER */
.summary-section {
  background: #fdfdfd;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  font-family: "Segoe UI", Roboto, sans-serif;
  color: #333;
  max-height: 600px;
  overflow-x: hidden;
  overflow-y: scroll;
  width: 100%;

  /* PRZYCISKI (Globalne) */
  button {
    cursor: pointer;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    transition: all 0.2s ease-in-out;
    display: inline-flex;
    align-items: center;
    justify-content: center;

    &:hover {
      filter: brightness(1.1);
      transform: translateY(-1px);
    }
    &:active {
      transform: translateY(0);
    }
  }

  /* LISTA PIZZ W TRAKCIE WYBORU */
  ul {
    list-style: none;
    padding: 0;
    margin: 15px 0;

    li {
      background: white;
      border: 1px solid #edf2f7;
      border-radius: 12px;
      padding: 10px 15px;
      margin-bottom: 8px;
      display: flex;
      align-items: center;
      gap: 10px;

      font-weight: 600;
      text-transform: uppercase;
      font-size: 0.85rem;
      color: #2d3436;

      /* Dropdown SELECT - Intuicyjny wygląd */
      select {
        margin-left: auto;
        padding: 6px 32px 6px 12px;
        border-radius: 8px;
        border: 2px solid #6c5ce7;
        background-color: #fff;
        color: #6c5ce7;
        font-weight: 800;
        appearance: none;
        cursor: pointer;
        background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236c5ce7' stroke-width='3' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
        background-repeat: no-repeat;
        background-position: right 10px center;
        background-size: 12px;

        &:hover {
          background-color: #f0eeff;
        }
      }
    }
  }

  /* WIDOK ZATWIERDZONEGO ZAMÓWIENIA (.orders-list) */
  .orders-list {
    margin-top: 30px;
    padding-top: 20px;
    border-top: 2px dashed #dfe6e9;

    h3 {
      color: #2d3436;
      font-size: 1.2rem;
      margin-bottom: 15px;
    }

    .ordered-pizzas-summary {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 20px;

      li {
        background: #f1f2f6;
        border: none;
        padding: 8px 12px;
        border-radius: 20px;
        font-size: 0.9rem;
        text-transform: none;

        strong {
          color: #6c5ce7;
          margin-left: 5px;
        }
      }
    }

    /* Karta osoby płacącej */
    .final-order-card {
      background: #f8faff;
      border: 2px solid #6c5ce7;
      border-radius: 14px;
      padding: 20px;
      margin-bottom: 25px;
      box-shadow: 0 4px 12px rgba(108, 92, 231, 0.1);

      .payer-info {
        font-size: 1.1rem;
        margin: 0 0 10px 0;
        strong {
          color: #6c5ce7;
          font-size: 1.2rem;
        }
      }

      .phone-info {
        margin: 0 0 15px 0;
        .phone-link {
          color: #0984e3;
          text-decoration: none;
          font-weight: bold;
          border-bottom: 1px solid transparent;
          &:hover {
            border-bottom-color: #0984e3;
          }
        }
      }

      .total-price-display {
        font-size: 1.3rem;
        margin: 0;
        padding-top: 10px;
        border-top: 1px solid #dcdde1;
        strong {
          color: #2ecc71;
        }
      }
    }

    /* Lista kto ile oddaje */
    .participant-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 12px 15px;
      background: white;
      border: 1px solid #edf2f7;
      border-radius: 10px;
      margin-bottom: 8px;

      .person-details {
        font-size: 0.95rem;
        color: #636e72;
      }

      .person-amount {
        color: #d63031;
        font-size: 1.05rem;
      }
    }
  }

  /* WIDOK MANAGERA I FORMULARZ */
  .MenagerView {
    display: block;
    background: #2d3436;
    color: white;
    padding: 20px;
    border-radius: 12px;
    margin-top: 20px;

    span {
      color: #00cec9;
      font-weight: bold;
    }

    button {
      background: #6c5ce7;
      color: white;
      width: 100%;
      padding: 14px;
      margin-top: 15px;
      font-size: 1rem;
    }
  }

  /* ELEMENTY DODATKOWE */
  .divider {
    border: none;
    height: 1px;
    background: #dfe6e9;
    margin: 30px 0;
  }

  .empty-state {
    text-align: center;
    padding: 30px;
    color: #b2bec3;
    font-style: italic;
  }

  .admin-buttons {
    display: flex;
    gap: 10px;

    .btn-refresh {
      background: #00cec9;
      color: #2d3436;
      flex: 2;
    }

    .btn-clear {
      background: #ff7675;
      color: white;
      flex: 1;
    }

    button {
      padding: 12px;
    }
  }

  .toggle-notes {
    background: #6c5ce7;
    color: white;
    padding: 10px 20px;
    margin-bottom: 20px;
  }

  /* DODAJ LUB ZAMIEŃ W .summary-section */
  .order-header {
    display: flex;
    flex-direction: column; // Układ pionowy dla lepszej czytelności
    gap: 12px;
    margin-bottom: 15px;

    .user-info {
      display: flex;
      align-items: center;
      gap: 10px;

      .customer-name {
        font-size: 1.1rem;
        font-weight: 800;
        color: #2d3436;
      }

      .counter-btns {
        display: flex;
        gap: 5px;
        margin-left: auto;

        button {
          width: 28px;
          height: 28px;
          border-radius: 50%;
          background: #f1f2f6;
          color: #6c5ce7;
          border: 1px solid #dfe6e9;
          font-size: 1rem;

          &:hover {
            background: #6c5ce7;
            color: white;
          }
        }
      }
    }

    .price-info {
      background: #f8f9fa;
      padding: 10px;
      border-radius: 8px;
      position: relative;

      p {
        margin: 2px 0;
        font-size: 0.9rem;
      }

      .slices-count strong {
        color: #6c5ce7;
      }

      .amount {
        font-weight: 600;
        span {
          color: #2ecc71; // Zielony kolor dla ceny
          font-size: 1.1rem;
        }
      }

      .restaurant-tag {
        position: absolute;
        top: 10px;
        right: 10px;
        font-size: 0.7rem;
        background: #dfe6e9;
        padding: 2px 8px;
        border-radius: 4px;
        text-transform: uppercase;
        color: #636e72;
      }
    }
  }

  /* STYLIZACJA INPUTA NUMERU TELEFONU */
  .phone-number-paying-person {
    font-size: 1rem;
    color: #00cec9; // Turkusowy pasujący do akcentów Managera
    margin-bottom: 8px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  input[type="tel"] {
    width: 100%;
    padding: 12px 16px;
    font-size: 1.1rem;
    font-family: inherit;
    color: #2d3436;
    background-color: #ffffff;
    border: 2px solid #dfe6e9;
    border-radius: 10px;
    outline: none;
    transition: all 0.3s ease;
    box-sizing: border-box; // Ważne, żeby padding nie rozpychał szerokości
    margin-bottom: 15px;

    &::placeholder {
      color: #b2bec3;
      letter-spacing: 2px;
    }

    &:focus {
      border-color: #6c5ce7; // Fioletowy focus
      box-shadow: 0 0 0 4px rgba(108, 92, 231, 0.15);
      background-color: #fff;
    }

    /* Styl dla Managera - jeśli input jest w ciemnym boksie */
    .MenagerView & {
      background-color: #3d4446;
      border-color: #4a4a4a;
      color: white;

      &:focus {
        border-color: #00cec9;
        box-shadow: 0 0 0 4px rgba(0, 206, 201, 0.2);
      }
    }
  }
}
</style>
