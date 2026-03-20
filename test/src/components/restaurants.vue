<template>
  <div class="menu-container">
    <header class="menu-header">
      <h2 class="menu-title">Wybierz miejsce</h2>
    </header>

    <div class="restaurants">
      <div
        class="restaurants__item"
        v-for="(restaurant, key) in restaurants"
        :key="key"
        @click="select(key)"
      >
        <div class="item-border">
          <span class="restaurant-name">{{ restaurant.name }}</span>
          <span class="view-menu">Zobacz kartę</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ChoseRestaurant",
  props: {
    restaurants: {
      type: Array,
      default: () => [],
    },
  },
  methods: {
    select(key) {
      this.$emit("select", key);
    },
  },
};
</script>

<style lang="scss" scoped>
.menu-container {
  background-color: transparent;
  padding: 3rem 1rem;
  font-family: "Inter", sans-serif;
  color: #1a1a1a;
}

.menu-header {
  text-align: left;
  margin-bottom: 3.5rem;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  border-left: 8px solid #d9534f; /* Grubszy akcent */
  padding-left: 2rem;

  .menu-title {
    font-size: 2.8rem; /* Większy tytuł sekcji */
    margin: 0;
    font-weight: 900;
    letter-spacing: -1.5px;
    color: #000;
    text-transform: uppercase;
  }
}

.restaurants {
  display: grid;
  /* Zwiększyłem min-width na 350px, żeby bloczki były solidniejsze */
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto;

  &__item {
    background: white;
    cursor: pointer;
    transition: all 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
    border-radius: 20px; /* Większe zaokrąglenie */
    border: 1px solid #e9ecef;
    overflow: hidden;
    position: relative;
    min-height: 250px; /* WYŻSZE BLOCZKI */
    display: flex;

    .item-border {
      padding: 2rem;
      display: flex;
      flex-direction: column;
      align-items: center; /* TEKST NA ŚRODKU POZIOMO */
      justify-content: center; /* TEKST NA ŚRODKU PIONOWO */
      width: 100%;
      height: 100%;
      text-align: center;
    }

    .restaurant-name {
      font-size: 2rem; /* DUŻA CZCIONKA NAZWY */
      font-weight: 900;
      margin-bottom: 1rem;
      color: #000;
      line-height: 1.1;
      transition: transform 0.3s ease;
    }

    .view-menu {
      font-size: 1rem;
      font-weight: 800;
      color: #d9534f;
      text-transform: uppercase;
      letter-spacing: 1px;
      display: flex;
      align-items: center;
      gap: 10px;

      &::after {
        content: "→";
        font-size: 1.4rem;
        transition: transform 0.3s;
      }
    }

    &:hover {
      transform: translateY(-12px); /* Mocniejszy skok do góry */
      border-color: #1a1a1a;
      box-shadow: 0 25px 50px rgba(0, 0, 0, 0.12);

      .restaurant-name {
        color: #d9534f;
        transform: scale(1.05); /* Lekkie powiększenie nazwy przy hoverze */
      }

      .view-menu::after {
        transform: translateX(8px);
      }
    }

    /* Gruby pasek dekoracyjny na górze */
    &::before {
      content: "";
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 8px; /* Grubszy pasek */
      background: #1a1a1a;
      transform: scaleX(0);
      transition: transform 0.3s;
      transform-origin: center; /* Animacja od środka */
    }

    &:hover::before {
      transform: scaleX(1);
    }
  }
}
</style>
