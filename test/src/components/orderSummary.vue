<template>
  <div
    class="col summary-section"
    :class="{ 'is-collapsed': !isNotesExpanded }"
  >
    <button class="toggle-notes" @click="isNotesExpanded = !isNotesExpanded">
      {{ isNotesExpanded ? "✕ Ukryj" : "☰ Pokaż zamówienia" }}
    </button>
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
                <span class="customer-name">
                  <input
                    type="radio"
                    v-model="payingPerson"
                    :value="item.name"
                  />
                  {{ item?.name }}

                  <button @click="updateSlice(item, 1)" title="Dodaj kawałek">
                    +
                  </button>
                  <button
                    @click="updateSlice(item, -1)"
                    title="Odejmij kawałek"
                  >
                    -
                  </button>
                </span>
                <span>
                  zje {{ item.slice }} kawałki
                  {{ getPlural(item.slice, ["kawałek", "kawałki", "kawałków"])
                  }}<br />
                  Ma do zapłacenia
                  {{ calculatePrice(item) }}
                </span>
                <span class="restaurant-name">{{
                  item?.selectedRestaurant || "Nieznana"
                }}</span>
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
          <div v-if="isCurrentUserManager">

          </div>
          <div  v-for="(pizzas, restaurant) in SummaryOrders" :key="restaurant" >
            <h3>{{ restaurant }}</h3>
            <!-- {{ selectPizzas }} -->
            <ul v-if="isCurrentUserManager">
              <li v-for="(count, name) in pizzas" :key="name">
                <input
                  type="checkbox"
                  :value="name"
                  v-model="checkedPizza"
                  
                />
                {{ name }}: <span v-if="count < 2">{{ count }}</span>
                <!-- ten v-model albo nie działa albo nie jest potrzbeny -->
                <select v-if="count > 1" v-model.number="selectPizzas[name]">
                  <option v-for="n in count" :key="n" :value="n">
                    {{ n }}
                  </option>
                </select>
                szt.
              </li>
            </ul>
          </div>
          <strong v-if="isCurrentUserManager">
            Łączna liczba kawałków {{ SummarySlices }} łączna cena
            <span>{{ TotalMoney }}</span>
            <div v-if="payingPerson">
              <div v-if="payingPerson" class="paying-info">
                Płaci: <strong>{{ payingPerson }}</strong>
                <div>
                  <h2>Podaj numer telefonu</h2>
                  <input
                    type="tel"
                    name="tel"
                    id="tel"
                    v-model="tel"
                    placeholder="000-000-000"
                  />
                  <p>Numer: {{ tel }}</p>
                  <!-- <button @click="AddPhoneNumer(tel)">Dodaj</button> -->
                </div>
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
                🍕 <strong>{{ count }}</strong>
              </span>
            </li>
          </ul>

          <div class="final-order-card">
            <p class="payer-info">
              💰 Płaci: <strong>{{ order.payer.name }}</strong>
            </p>
            <p class="phone-info">
              📞 Tel:
              <a :href="'tel:' + order.payer.phone" class="phone-link">{{
                order.payer.phone
              }}</a>
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
            <span class="person-details"
              >👤 {{ person.name }} ({{ person.slices }} szt.)</span
            >
            <strong class="person-amount">{{ person.toPay }} zł</strong>
          </div>
        </div>
      </div>

      <div v-else class="empty-state">
        <p>Brak danych lub czekam na zatwierdzenie...</p>
      </div>

      <hr class="divider" />

      <div class="admin-buttons">
        <button class="btn-refresh" @click="fetchOrderedPizzas">
          Pokaż całe zamówienie
        </button>
        <button class="btn-clear" @click="clearApi">Wyczyść API</button>
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
      FullOrderFlag: true,
      tel: "",
      steps: [
        { valid: false }, // Krok 0
        { valid: false }, // Krok 1
        { valid: false }, // Krok 2 (ten, którego szukał watcher)
      ],
    };
  },
  watch: {
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
      // potrzebne
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
    async fetchSummary() {
      //Potrzebne
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

    //CHWILOWE CZYSZCENIE
    async clearApi() {
      try {
        const clearResponse = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=clearTodayOrder`,
        );

        if (clearResponse.ok) {
          console.log("jeje");
        }
      } catch (error) {
        console.error(error);
      }
    },
    // iovniosdnviosdnvdsinvsdiondinvidosnvdinvisodnvisdnvoinvisonvdino
    async updateSlice(item, change) {
      const newCount = (parseFloat(item.slice) || 0) + change;
      item.slice = newCount;
      this.UpdataOrder(item);
    },
    async approveFlavors() {
      const hasPizzas = Object.keys(this.checkedPizza || {}).length > 0;

      if (!hasPizzas) {
        alert("Najpierw wybierz jakieś pizze do zamówienia!");
        return;
      }

      const person = this.summary.find(
        (p) => p && p.name === this.payingPerson,
      );

      const phoneToVerify = this.tel || (person ? person.phone : null);

      if (!this.payingPerson || !phoneToVerify) {
        alert("Musisz wybrać osobę zamawiającą i podać numer telefonu!");
        return;
      }

      // 3. Opcjonalna walidacja formatu telefonu (np. minimum 9 cyfr)
      if (phoneToVerify.toString().replace(/\s/g, "").length < 9) {
        alert("Numer telefonu wydaje się być za krótki!");
        return;
      }

      // Reszta Twojej logiki zapisu...
      try {
        const clearResponse = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=clearTodayOrder`,
        );

        if (clearResponse.ok) {
          const participantsList = this.summary
            .filter((item) => item && item.name)
            .map((item) => ({
              name: item.name,
              slices: item.slice,
              toPay: this.calculatePrice(item),
              restaurant: item.selectedRestaurant,
            }));

          this.FinalOrderSummary = {
            selectedPizzas: { ...this.checkedPizza },
            payer: {
              name: this.payingPerson,
              phone: phoneToVerify, // Używamy zweryfikowanego numeru
              totalToPayAtRestaurant: this.TotalMoney,
            },
            participants: participantsList,
          };

          // Wysyłka POST (tak jak w Twoim kodzie)
          const setResponse = await fetch(
            `https://webwizards.home.pl/jacek/pizza/api/?method=setTodayOrder`,
            {
              method: "POST",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify(this.FinalOrderSummary),
            },
          );

          if (setResponse.ok) {
            alert("Zamówienie i lista składek zostały zapisane!");
          }
        }
      } catch (error) {
        console.error("Błąd podczas procesowania zamówienia:", error);
      }
    },
    async fetchOrderedPizzas() {
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getTodayOrder`,
        );
        const data = await response.json();

        this.orderedPizzas = data;
      } catch (error) {
        console.error("Błąd pobierania:", error);
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
      const pizzas = this.selectPizzas || {};
      const entries = Object.entries(pizzas);
      console.log(entries);
      return "";
    },
    calculatePrice(item) {
      const totalCost = parseFloat(this.TotalMoney) || 0;
      const totalSlices = this.totalSlicesCount;
      if (totalCost === 0 || totalSlices === 0) return "0.00";
      const pricePerSlice = totalCost / totalSlices;
      return (pricePerSlice * (parseInt(item.slice) || 0)).toFixed(2);
    },

    WhoPaied() {
      const TelNumber = this.summary.find((p) => p && p.tel == p.tel);
      return TelNumber;
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

            // TO JEST KLUCZOWE:
            // Jeśli tej pizzy nie ma jeszcze w selectPizzas, dodaj ją z wartością 1
            if (this.selectPizzas[pName] === undefined) {
              this.selectPizzas[pName] = 1;
            }

            totals[restaurantKey][pName] =
              (totals[restaurantKey][pName] || 0) + 1;
          });
        }
      });

      return totals;
    },

    isCurrentUserManager() {
      const savedName = Cookies.get("user_name");
      if (!savedName || !this.summary) return false;

      // Szukamy osoby o tym imieniu w liście uczestników
      const currentUser = this.summary.find((p) => p && p.name === savedName);

      // Zwraca true tylko, jeśli osoba istnieje i ma manager: true
      return currentUser && currentUser.manager === true;
    },

    isFullOrderVisible() {
      if (!this.summary || !Array.isArray(this.summary)) return false;
      return this.summary.some((item) => item && item.name);
    },
    TotalMoney() {
      let total = 0;
      if (!this.checkedPizza || this.checkedPizza.length === 0) return "0.00";

      // Iterujemy po zaznaczonych nazwach pizz do zamówienia
      this.checkedPizza.forEach((pizzaName) => {
        // Pobieramy ilość z Twojego selecta (domyślnie 1, jeśli nie wybrano inaczej)
        const quantity = parseInt(this.selectPizzas[pizzaName]) || 1;

        // Musimy znaleźć cenę tej pizzy. Szukamy jej w menu pierwszej
        // lepszej restauracji, która ją oferuje
        let price = 0;
        for (const restaurant in this.allMenus) {
          const pizzaInMenu = this.allMenus[restaurant].find(
            (p) => p.name === pizzaName,
          );
          if (pizzaInMenu) {
            // Pobieramy cenę (zakładam, że variants[1] to ta właściwa cena)
            price = pizzaInMenu.variants?.[1]?.price || 0;
            break;
          }
        }

        total += price * quantity;
      });

      return total.toFixed(2);
    },
    SlicePrice() {
      return 0;
    },

    SummarySlices() {
      if (!this.summary || !Array.isArray(this.summary)) return "0";

      const totalWantedSlices = this.summary.reduce((total, item) => {
        return total + (parseInt(item?.slice) || 0);
      }, 0);

      let pizzasSelected = 0;
      this.checkedPizza.forEach((pName) => {
        pizzasSelected += parseInt(this.selectPizzas[pName]) || 1;
      });
      const remainder = totalWantedSlices % 8;
      const missingToFull = remainder === 0 ? 0 : 8 - remainder;
      const targetSlices = totalWantedSlices + missingToFull;
      const targetPizzas = targetSlices / 8;

      const diffPizzas = targetPizzas - pizzasSelected;
      if (diffPizzas > 0) {
        const plural =
          diffPizzas === 1 ? "pizzę" : diffPizzas < 5 ? "pizze" : "pizz";
        let msg = `${totalWantedSlices} kawałków (brakuje ${missingToFull} do pełnej pizzy). `;
        if (missingToFull == 0) {
          msg = `to ${totalWantedSlices} `;
        }
        msg += `Musisz wybrać jeszcze ${diffPizzas} ${plural}.`;
        return msg;
      } else if (diffPizzas === 0) {
        return `${totalWantedSlices} kawałków. Idealnie! Wybrałeś odpowiednią ilość (${pizzasSelected} szt).`;
      } else {
        const over = Math.abs(diffPizzas);
        const plural = over === 1 ? "pizzę" : over < 5 ? "pizze" : "pizz";
        return `${totalWantedSlices} kawałków. Wybrałeś o ${over} ${plural} za dużo!`;
      }
    },

    // Dodaj to do computed:
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
  },

  mounted() {
    this.fetchrestaurants();
    this.fetchSummary();
    this.polling = setInterval(() => this.fetchSummary(), 3000);
  },
  beforeUnmount() {
    clearInterval(this.polling);
  },
};
</script>
<style scoped>
/* 1. KONTENER GŁÓWNY - Efekt kartki z zeszytu */
.summary-section {
  position: sticky;
  top: 90px;
  width: 450px;
  background: #fef9e7;
  padding: 30px;
  border: 1px solid #d1d1d1;
  font-family: "Inter", sans-serif;
  color: #1a1a1a;
  z-index: 10;
  transition: all 0.3s ease;
  box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.05);
}

/* Stan zwinięty */
.is-collapsed {
  height: 55px;
  width: 300px;
  overflow: hidden;
  padding-bottom: 0;
}

/* Czerwona linia marginesu */
.summary-section::after {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 35px;
  width: 1px;
  background: rgba(255, 0, 0, 0.2);
}

/* 2. TYPOGRAFIA I NAGŁÓWKI */
h2 {
  text-align: center;
  font-size: 1.1rem;
  font-weight: 900;
  text-transform: uppercase;
  border-bottom: 2px solid #1a1a1a;
  margin-bottom: 20px;
  padding-bottom: 8px;
}

h3,
h4 {
  font-size: 1rem;
  font-weight: 800;
  margin: 25px 0 10px 45px;
  text-transform: uppercase;
  color: #1a1a1a;
}

h4 {
  font-size: 0.85rem;
  color: #444;
}

/* 3. LISTY I STRUKTURA (Wcięcie 45px dla wszystkiego za linią) */
.order-list {
  max-height: 50vh;
  overflow-y: auto;
  padding-right: 5px;
}

.order-list::-webkit-scrollbar {
  width: 6px;
}
.order-list::-webkit-scrollbar-thumb {
  background: #1a1a1a;
  border-radius: 10px;
}

ul {
  list-style: none;
  padding: 0;
}

.order-card {
  padding: 15px 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

/* Globalne wcięcie dla elementów tekstowych */
.restaurant-name,
.pizza-list,
.order-header span:nth-child(2),
.paying-user-badge,
.participant-row,
.final-order-card,
.ordered-pizzas-summary,
.empty-state,
.admin-buttons,
.debug-info {
  margin-left: 45px;
  display: block;
  line-height: 1.6;
}

/* 4. ELEMENTY INTERAKTYWNE */
.customer-name {
  display: flex;
  align-items: center;
  gap: 12px;
  font-weight: 900;
  font-size: 1.1rem;
}

.paying-user-badge {
  display: flex;
  align-items: center;
  gap: 10px;
  background: #fff;
  border: 2px solid #1a1a1a;
  padding: 10px 15px;
  margin-top: 15px;
  margin-bottom: 15px;
  font-weight: 900;
  box-shadow: 4px 4px 0px #1a1a1a;
}

.paying-user-badge::before {
  content: "💸";
}

select {
  font-family: inherit;
  font-weight: 900;
  border: 2px solid #1a1a1a;
  border-radius: 4px;
  padding: 2px 6px;
  background: #fff;
  cursor: pointer;
}

/* 5. PRZYCISKI I INPUTY */
button {
  cursor: pointer;
  background: #1a1a1a;
  color: #fff;
  border: none;
  border-radius: 4px;
  font-weight: 800;
  padding: 8px 16px;
  transition: 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

button:hover {
  background: #d9534f !important;
  transform: translateY(-2px);
}

.toggle-notes {
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 4px 10px;
  font-size: 0.65rem;
  text-transform: uppercase;
}

.customer-name button {
  width: 26px;
  height: 26px;
  padding: 0;
}

input[type="tel"] {
  font-weight: 900;
  font-size: 1.5rem;
  border: none;
  border-bottom: 3px solid #d9534f;
  width: 100%;
  background: transparent;
  outline: none;
  padding: 8px 0;
}

/* 6. PODSUMOWANIE FINANSOWE (Główny blok) */
strong {
  font-weight: 900;
}

.summary-section > strong {
  display: block;
  margin-top: 25px;
  padding: 20px;
  border: 3px solid #1a1a1a;
  background: #fff;
  text-align: center;
}

.summary-section > strong span {
  color: #d9534f;
  font-size: 1.6rem;
  display: block;
  margin-top: 5px;
}

/* 7. KARTA ZREALIZOWANEGO ZAMÓWIENIA (ordered-list) */
.final-order-card {
  background: #fff;
  border: 2px solid #feb2b2;
  padding: 15px;
  border-radius: 12px;
  margin: 15px 0 15px 45px;
  box-shadow: 4px 4px 0px #feb2b2;
}

.phone-link {
  color: #d9534f;
  text-decoration: none;
  font-weight: 800;
  border-bottom: 1px dashed #d9534f;
}

.total-price-display {
  font-size: 1.2rem;
  color: #e74c3c;
  margin-top: 10px;
}

.participant-row {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  font-size: 0.95rem;
}

/* 8. FORMULARZE: CHECKBOX I RADIO */
input[type="checkbox"],
input[type="radio"] {
  appearance: none;
  width: 20px;
  height: 20px;
  border: 2px solid #1a1a1a;
  background: #fff;
  cursor: pointer;
  position: relative;
  flex-shrink: 0;
}

input[type="radio"] {
  border-radius: 50%;
}
input[type="checkbox"]:checked,
input[type="radio"]:checked {
  background: #1a1a1a;
}
input[type="checkbox"]:checked::after {
  content: "✕";
  position: absolute;
  color: #fff;
  font-size: 12px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.divider {
  border: none;
  border-top: 1px dashed #d1d1d1;
  margin: 25px 0;
}

.admin-buttons {
  display: flex;
  gap: 10px;
}
</style>
