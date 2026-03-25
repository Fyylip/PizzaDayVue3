<template>
  <div class="body">
    <topBar :steps="steps" :currentStep="currentStep" />
    <div class="content">
      <div class="col-main" v-if="currentStep !== 4">
        <div v-if="isFormVisible && !hasOrderedPizzas">
          <userForm v-if="currentStep === 1" v-on:set="user = $event" />
        </div>

        <div v-else-if="hasOrderedPizzas" class="finalized-alert">
          <h2>Zamówienie zamknięte 🍕</h2>
          <p>
            Ktoś już sfinalizował zamówienie. Sprawdź szczegóły w podsumowaniu.
          </p>
          <button @click="currentStep = 4">Przejdź do podsumowania</button>
        </div>

        <restaurants
          v-if="currentStep === 2 && !hasOrderedPizzas"
          :restaurants="restaurants"
          v-on:select="selectedRestaurant = $event"
        />

        <restaurantMenu
          v-if="currentStep === 3 && !hasOrderedPizzas"
          :menu="menu"
          :selected="pizzas"
          v-on:select="selectPizza($event)"
        />

        <div v-if="currentStep === 3 && !hasOrderedPizzas" class="actions">
          <button
            @click="
              submitOrder(order);
              currentStep = 4;
            "
            class="submit-btn"
          >
            Wyślij
          </button>
        </div>
      </div>

      <aside
        class="col-sidebar"
        :class="{ 'center-sidebar': currentStep === 4 }"
      >
        <OrderSummary />
      </aside>
    </div>

    <span style="font-size: 8px; color: gray; letter-spacing: 1px">
      Autor Filip Najlepszy Praktykant (:
    </span>
  </div>
</template>

<script>
import Cookies from "js-cookie";

import topBar from "./components/topBar.vue";
import userForm from "./components/userForm.vue";
import restaurants from "./components/restaurants.vue";
import restaurantMenu from "./components/restaurantMenu.vue";
import OrderSummary from "./components/orderSummary.vue";

export default {
  components: {
    topBar,
    userForm,
    restaurants,
    restaurantMenu,
    OrderSummary,
  },
  data() {
    return {
      currentStep: 1,
      selectedRestaurant: null,
      steps: [
        { name: "Podaj swoje dane", valid: false },
        { name: "Wybierz pizzerię", valid: false },
        { name: "Na co masz ochotę?", valid: false },
        { name: "Podsumowanie", valid: false },
      ],
      restaurants: [],
      menu: [],
      allMenus: {},
      pizzas: [],
      user: {},
      summary: [],
      isNotesExpanded: false,
    };
  },
  watch: {
    user: {
      handler() {
        if (this.user && Object.keys(this.user).length > 0) {
          this.steps[0].valid = true;
          this.currentStep = 2;
        }
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
    hasOrderedPizzas(newVal) {
      if (newVal) {
        this.currentStep = 4;
      }
    },
  },
  methods: {
    async fetchrestaurants() {
      try {
        const response = await fetch(
          "https://webwizards.home.pl/jacek/pizza/api/?method=getLocals",
        );
        const data = await response.json();
        this.restaurants = data;
      } catch (error) {
        console.error("Błąd pobierania:", error);
      }
    },
    async fetchMenu(key) {
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getMenu&pizzeria=${key}`,
        );
        const data = await response.json();
        this.menu = data;
        this.allMenus[key] = data;
      } catch (error) {
        console.error(error);
      }
    },
    getPizzaNameFromMenu(pizzaId, restaurantKey) {
      const targetMenu = this.allMenus[restaurantKey];
      if (targetMenu) {
        const pizza = targetMenu.find((p) => p.id == pizzaId);
        return pizza ? pizza.name : `Pizza (ID: ${pizzaId})`;
      } else {
        return `Ładowanie...`;
      }
    },
    async submitOrder(orderPayload) {
      try {
        await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=setTodayOrder`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(orderPayload),
          },
        );
        this.pizzas = [];
        this.currentStep = 4;
        window.location.reload();
      } catch (error) {
        console.error(error);
      }
    },
    async fetchSummary() {
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getTodayOrder`,
        );
        const data = await response.json();
        this.summary = data;
      } catch (error) {
        console.error(error);
      }
    },
    selectPizza(id) {
      if (this.pizzas.includes(id)) {
        this.pizzas = this.pizzas.filter((e) => e != id);
      } else {
        this.pizzas.push(id);
      }
    },
  },
  computed: {
    order() {
      return {
        ...this.user,
        pizzas: this.pizzas,
        selectedRestaurant: this.selectedRestaurant,
      };
    },
    hasOrderedPizzas() {
      if (!this.summary) return false;
      const summaryData = Array.isArray(this.summary)
        ? this.summary
        : [this.summary];
      const finalOrder = summaryData.find((item) => item && item.payer);
      if (!finalOrder || !finalOrder.selectedPizzas) return false;
      const pizzaNames = Object.keys(finalOrder.selectedPizzas).filter(
        (name) => name !== "Ładowanie...",
      );
      return pizzaNames.length > 0;
    },
    isFormVisible() {
      const savedName = Cookies.get("user_name");
      if (!savedName) return true;
      if (!this.summary || !Array.isArray(this.summary)) return true;
      const alreadyOrdered = this.summary.some((order) => {
        return order && order.name === savedName;
      });
      return !alreadyOrdered;
    },
  },
  mounted() {
    this.fetchrestaurants();
    this.fetchSummary().then(() => {
      if (this.hasOrderedPizzas) {
        this.currentStep = 4;
      } else if (!this.isFormVisible && this.currentStep === 1) {
        this.currentStep = 4;
      }
    });

    this.polling = setInterval(() => {
      this.fetchSummary();
    }, 5000);
  },
  beforeUnmount() {
    clearInterval(this.polling);
  },
};
</script>
<style scoped>
.body {
  margin: 0;
  padding: 0;
  padding-bottom: 100px; /* Miejsce na przyklejony przycisk, by nie zasłaniał treści */
  min-height: 100vh;
  background-color: #f8f9fa;
}

.content {
  display: flex;
  flex-direction: row;
  gap: 20px;
  padding: 40px;
  max-width: 1600px;
  margin: 0 auto;
  align-items: flex-start;
  box-sizing: border-box;
}

.col-main {
  flex: 1;
  min-width: 0;
  margin-right: 20px;
}

.col-sidebar {
  width: 480px;
  position: sticky;
  top: 100px;
  display: flex;
  justify-content: flex-start;
  padding-right: 20px;
  box-sizing: border-box;
}

/* Kontener przycisku - teraz jako stała belka na dole */
.actions {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  backdrop-filter: blur(2px);
  padding: 20px 0;
  display: flex;
  justify-content: center;
  z-index: 2000;
}

.submit-btn {
  /* Szerszy przycisk */
  width: 80%;
  max-width: 600px;

  color: #b22222;
  border: 4px solid #b22222;
  padding: 15px 0;
  font-family: "Inter", sans-serif;
  font-size: 1.5rem;
  font-weight: 900;
  text-transform: uppercase;
  cursor: pointer;
  background: white;
  transition: all 0.2s;
  /* Lekki obrót dla stylu pieczątki */
  transform: rotate(-1deg);
}

.submit-btn:hover {
  background: #b22222;
  color: white;
  transform: rotate(0deg) scale(1.02);
}

/* Styl dla zablokowanego przycisku */
.submit-btn:disabled {
  border-color: #ccc;
  color: #ccc;
  cursor: not-allowed;
  transform: none;
}

/* ... reszta Twoich stylów ... */

.col-sidebar {
  width: 480px;
  position: sticky;
  top: 100px;
  display: flex;
  justify-content: flex-start;
  padding-right: 20px;
  box-sizing: border-box;
  transition: all 0.5s ease; /* Płynne przejście do środka */
}

/* Nowa klasa do centrowania */
.center-sidebar {
  width: 100%;
  max-width: 800px; /* Możesz dostosować szerokość podsumowania końcowego */
  margin: 0 auto;
  position: static;
  padding-right: 0;
  justify-content: center;
}

/* Opcjonalnie: poprawienie kontenera content w kroku 4 */
.content:has(.center-sidebar) {
  justify-content: center;
}

@media (max-width: 1200px) {
  .content {
    flex-direction: column;
    align-items: center;
    padding: 20px;
  }

  .col-sidebar {
    width: 100%;
    max-width: 450px;
    position: static;
    order: 2;
    padding-right: 0;
    margin-top: 40px;
  }

  .col-main {
    width: 100%;
    margin-right: 0;
  }
}
</style>
