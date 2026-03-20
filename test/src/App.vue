<template>
  <div class="body">
    <topBar :steps="steps" :currentStep="currentStep" />

    <div class="content">
      <div class="col-main">
        <userForm v-if="currentStep === 1" v-on:set="user = $event" />

        <restaurants
          v-if="currentStep === 2"
          :restaurants="restaurants"
          v-on:select="selectedRestaurant = $event"
        />

        <restaurantMenu
          v-if="currentStep === 3"
          :menu="menu"
          :selected="pizzas"
          v-on:select="selectPizza($event)"
        />

        <div v-if="currentStep === 3" class="actions">
          <button
            @click="
              submitOrder(order);
              this.currentStep = 1;
            "
            class="submit-btn"
          >
            Wyślij 
          </button>
        </div>
      </div>

      <aside class="col-sidebar">
        <OrderSummary />
      </aside>

      <FinalOrder />
    </div>
    <span style="font-size: 8px; color: gray; letter-spacing: 1px"
      >Autor Filip Najlepszy Praktykant (:
    </span>
  </div>
</template>
<script>
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
      ],
      restaurants: [],
      menu: [],
      allMenus: {},
      pizzas: [],
      user: {},
      summary: null,
      isNotesExpanded: false,
    };
  },
  watch: {
    user: {
      handler() {
        this.steps[0].valid = true;
        this.currentStep = 2;
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
        this.fetchMenuToCache(restaurantKey);
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
        //jjsjsjdjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjj
        // this.fetchSummary();
        window.location.reload()
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
  },
  mounted() {
    this.fetchrestaurants();
  },
};
</script>
<style scoped>
.body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background-color: #f8f9fa; /* Jasne tło, o którym pisaliśmy */
}

.content {
  display: flex;
  flex-direction: row;
  /* Zmniejszyłem gap, by zyskać miejsce na szeroką notatkę */
  gap: 20px;
  padding: 40px;
  /* Zwiększyłem max-width, żeby przy szerokich ekranach wszystko się mieściło */
  max-width: 1600px;
  margin: 0 auto;
  align-items: flex-start;
  box-sizing: border-box;
}

/* Główna kolumna z lewej */
.col-main {
  flex: 1;
  min-width: 0;
  /* Dodatkowy margines, by odsunąć formularz od notatki */
  margin-right: 20px;
}

/* Kolumna z notatką - KLUCZOWE ZMIANY */
.col-sidebar {
  /* Szerokość dopasowana do nowej notatki (450px) + zapas na obrót (rotate) */
  width: 480px;
  position: sticky;
  top: 100px; /* Miejsce na topBar */
  display: flex;
  justify-content: flex-start;
  /* Zapobiega wyjeżdżaniu poza ekran w prawo */
  padding-right: 20px;
  box-sizing: border-box;
}

/* Przycisk pieczątka */
.actions {
  margin-top: 30px;
  display: flex;
  justify-content: center;
}

.submit-btn {
  position: fixed;
  bottom: 30px;
  right: 30px;
  z-index: 1000;
  color: #b22222;
  border: 4px solid #b22222;
  padding: 15px 40px;
  font-family: "Inter", sans-serif; /* Zmienione na normalną czcionkę */
  font-size: 1.3rem;
  font-weight: 900;
  text-transform: uppercase;
  cursor: pointer;
  transform: rotate(-2deg);
  background: white;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  transition: all 0.2s;
}

.submit-btn:hover {
  background: #fdf2f2;
  transform: rotate(0deg) scale(1.05);
}

/* Responsywność - poprawiona dla szerokiej notatki */
@media (max-width: 1200px) {
  .content {
    flex-direction: column;
    align-items: center;
    padding: 20px;
  }

  .col-sidebar {
    width: 100%;
    max-width: 450px; /* Na mobile niech nie będzie szersza niż notatka */
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
