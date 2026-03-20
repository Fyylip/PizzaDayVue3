<template>
  <div class="menu-wrapper">
    <div class="menu">
      <header class="menu__header">
        <h2 class="menu__title">Karta Dań</h2>
        <div class="menu__line"></div>
      </header>

      <div
        :class="[
          'menu__item',
          { 'menu__item--selected': selected.includes(item.id) },
        ]"
        v-for="item in menu"
        :key="item.id"
        @click="$emit('select', item.id)"
      >
        <div class="menu__content">
          <div class="menu__info">
            <strong class="menu__name">{{ item.name }}</strong>
            <p class="menu__ingredients">{{ item.ingredients }}</p>
            <!-- <span class="price">{{ item.variants[1]?.price }} zł</span> -->
          </div>

          <div class="menu__action">
            <button
              :class="[
                'select-button',
                { 'select-button--active': selected.includes(item.id) },
              ]"
            >
              {{ selected.includes(item.id) ? "Wybrano" : "Mam ochotę" }}
              <span class="icon">{{
                selected.includes(item.id) ? "✓" : "+"
              }}</span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    menu: {
      default: () => [],
    },

    selected: {
      default: () => [],
    },
  },
};
</script>

<style lang="scss" scoped>
/* Używamy tylko Inter dla spójności z resztą aplikacji Filipa */
@import url("https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap");

.menu-wrapper {
  padding: 2rem;
  background-color: transparent; /* Tło bierze z App.vue */
  display: flex;
  justify-content: center;
}

.menu {
  width: 100%;
  max-width: 800px;
  background: #ffffff;
  padding: 2rem;
  border-radius: 16px; /* Nowoczesne zaokrąglenie */
  border: 1px solid #e9ecef;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.05);
  font-family: "Inter", sans-serif;

  &__header {
    text-align: left; /* Bardziej nowoczesne wyrównanie do lewej */
    margin-bottom: 2rem;
    padding-left: 1rem;
    border-left: 5px solid #d9534f; /* Czerwony akcent jak przy telefonie */
  }

  &__title {
    font-size: 1.8rem;
    font-weight: 800;
    color: #1a1a1a;
    margin: 0;
    letter-spacing: -0.5px;
  }

  &__item {
    border-bottom: 1px solid #f1f3f5;
    padding: 1.2rem 1rem;
    transition: all 0.2s ease-in-out;
    cursor: pointer;
    border-radius: 12px;
    margin-bottom: 4px;

    &:hover {
      background-color: #f8f9fa;
      transform: translateX(5px); /* Delikatne wysunięcie */
    }

    &--selected {
      background-color: #fff5f5; /* Bardzo jasny czerwony */
      border-color: #d9534f;

      &:hover {
        background-color: #fff0f0;
      }
    }
  }

  &__content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 20px;
  }

  &__name {
    display: block;
    font-size: 1.1rem;
    font-weight: 700;
    color: #000;
    margin-bottom: 4px;
  }

  &__ingredients {
    font-size: 0.9rem;
    color: #666;
    font-weight: 400;
    margin: 0;
    line-height: 1.4;
  }

  /* Nowoczesny przycisk akcji */
  .select-button {
    border: 2px solid #e9ecef;
    background: #fff;
    color: #444;
    padding: 10px 20px;
    border-radius: 10px;
    font-size: 0.85rem;
    font-weight: 700;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 10px;
    transition: all 0.2s;
    min-width: 140px;
    justify-content: center;

    .icon {
      font-size: 1.2rem;
    }

    &:hover {
      border-color: #1a1a1a;
      color: #1a1a1a;
    }

    &--active {
      background: #1a1a1a;
      border-color: #1a1a1a;
      color: white;

      &:hover {
        background: #d9534f; /* Kolor błędu/usuwania */
        border-color: #d9534f;
      }
    }
  }
}
</style>
