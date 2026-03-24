<template>
  <div class="form-container">
    <div class="form-card">
      <h3 class="form-title">Dane zamówienia</h3>

      <div class="input-group">
        <label for="name">Imię i nazwisko</label>
        <input
          v-model="form.name"
          type="text"
          id="name"
          placeholder="Jan Kowalski"
        />
      </div>

      <div class="input-group">
        <label for="slices">Ile zjesz kawałków</label>
        <select v-model="form.slice" id="slices">
          <option :value="null" disabled>Wybierz liczbę...</option>
          <option v-for="value in slicesOptions" :key="value" :value="value">
            {{ value }}
          </option>
        </select>
      </div>

      <div class="input-group">
        <label for="paied">Metoda płatności</label>
        <select v-model="form.paied" id="paied">
          <option value="" disabled>Wybierz płatność...</option>
          <option value="blik">Blik</option>
          <option value="cash">Gotówka</option>
        </select>
      </div>

      <div class="checkbox-section">
        <label class="custom-checkbox">
          <input type="checkbox" v-model="form.manager" />
          <span class="checkmark"></span>
          <span class="label-text">Ja będę zamawiał</span>
        </label>
      </div>

      <button class="submit-btn" @click="handleSubmit">Dalej</button>
    </div>
  </div>
</template>
<script>
import Cookies from "js-cookie";

export default {
  name: "UserForm",
  data() {
    return {
      summary: [],
      form: {
        name: "",
        paied: "",
        slice: null,
        manager: false,
        isPaingPerson: false,
      },
      slicesOptions: [1, 2, 3, 4, 5, 6, 7, 8],
      errors: {},
    };
  },

  async mounted() {
    await this.getTodayOrder();

    const savedName = Cookies.get("user_name");
    if (savedName) {
      this.form.name = savedName;
    }
  },

  methods: {
    async getTodayOrder() {
      try {
        const response = await fetch(
          `https://webwizards.home.pl/jacek/pizza/api/?method=getTodayOrder`,
        );
        const data = await response.json();

        if (data && data.payer) {
          this.isOrderFinalized = true;
        } else {
          this.isOrderFinalized = false;
        }

        this.summary = data.participants || (Array.isArray(data) ? data : []);
      } catch (error) {
        console.error("Błąd pobierania zamówień:", error);
        this.summary = [];
      }
    },

    validateForm() {
      const isNameValid = !!this.form.name && this.form.name.trim().length >= 3;
      const isSliceValid = !!this.form.slice;
      const isPaymentValid = !!this.form.paied;

      if (!isNameValid || !isSliceValid || !isPaymentValid) {
        alert("Przerósł Cię formularz. Wypełnij wszystkie pola!");
        return false;
      }

      const normalizedName = this.form.name.trim().toLowerCase();

      // 1. Sprawdzenie czy użytkownik już istnieje
      const alreadyExists = this.summary.some(
        (p) => p && p.name && p.name.trim().toLowerCase() === normalizedName,
      );

      if (alreadyExists) {
        alert(`Użytkownik o tym imieniu jest już na liście.`);
        return false;
      }

      // 2. NOWA WALIDACJA: Sprawdzenie czy jest już manager
      if (this.form.manager) {
        const hasManager = this.summary.some((p) => p && p.manager === true);

        if (hasManager) {
          alert(
            "Już ktoś inny jest menagerem dorośli jesteście zacznijcie się dogadywać XD",
          );
          this.form.manager = false;
          return false;
        }
      }

      return true;
    },

    handleSubmit() {
      if (this.validateForm()) {
        this.handleName();
        this.$emit("set", { ...this.form });
        console.log("Formularz wysłany!");
      } else {
        console.log("Formularz zawiera błędy.");
      }
    },

    handleName() {
      if (this.form.name && this.form.name.trim() !== "") {
        Cookies.set("user_name", this.form.name.trim(), { expires: 100 });
      }
    },
  },
};
</script>
<style lang="scss" scoped>
// Zmienne dla spójności
$pencil-color: #2d3436;
$accent-red: #e63946;
$paper-white: #ffffff;
$line-color: #b2bec3;
$font-stack: "Courier New", Courier, monospace;

.form-container {
  display: flex;
  justify-content: center;
  padding: 20px;
  background: none; // Usunięte tło kontenera
}

.form-card {
  background: $paper-white;
  padding: 40px 30px;
  max-width: 400px;
  width: 100%;
  position: relative;
  font-family: $font-stack;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.05);
  border: 1px solid #eee;

  // Efekt "wyrwanej kartki" (ząbki)
  &::before {
    content: "";
    position: absolute;
    top: -10px;
    left: 0;
    right: 0;
    height: 12px;
    background-image: radial-gradient(
      circle,
      transparent 50%,
      $paper-white 55%
    );
    background-size: 20px 20px;
    transform: translateY(-2px);
  }

  .form-title {
    text-align: center;
    margin-bottom: 35px;
    color: $pencil-color;
    text-transform: uppercase;
    border-bottom: 2px solid $accent-red;
    display: block;
    padding-bottom: 5px;
  }
}

.input-group {
  margin-bottom: 25px;

  label {
    display: block;
    font-size: 0.7rem;
    font-weight: bold;
    color: lighten($pencil-color, 20%);
    text-transform: uppercase;
    margin-bottom: 4px;
  }

  input,
  select {
    width: 100%;
    padding: 10px 0;
    border: none;
    border-bottom: 1px dashed $line-color;
    background: transparent;
    font-family: inherit;
    font-size: 1.1rem;
    outline: none;
    transition: border-color 0.3s;

    &:focus {
      border-bottom-color: $pencil-color;
      border-bottom-style: solid;
    }
  }
}

// --- NOWY STYL CHECKBOXA ---
.checkbox-section {
  margin: 30px 0;

  .custom-checkbox {
    display: flex;
    align-items: center;
    cursor: pointer;
    font-size: 0.9rem;
    user-select: none;

    input {
      position: absolute;
      opacity: 0; // Ukrywamy stary input
      cursor: pointer;
    }

    .checkmark {
      height: 20px;
      width: 20px;
      border: 2px solid $pencil-color;
      margin-right: 12px;
      position: relative;
      display: inline-block;
      flex-shrink: 0;

      // "X" widoczny po zaznaczeniu
      &::after {
        content: "X";
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        color: $accent-red;
        font-weight: bold;
        font-size: 18px;
        display: none;
      }
    }

    // Pokazujemy "X" gdy input jest zaznaczony
    input:checked ~ .checkmark::after {
      display: block;
    }

    &:hover .checkmark {
      background-color: rgba($line-color, 0.1);
    }

    .label-text {
      color: $pencil-color;
      font-weight: bold;
    }
  }
}

.submit-btn {
  width: 100%;
  padding: 15px;
  background-color: $pencil-color;
  color: white;
  border: none;
  font-family: inherit;
  font-weight: bold;
  text-transform: uppercase;
  cursor: pointer;
  transition: transform 0.1s;

  &:hover {
    filter: brightness(1.2);
  }

  &:active {
    transform: scale(0.98);
  }
}
</style>
