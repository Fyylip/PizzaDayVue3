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
          :class="{ 'input-error': errors.name }"
          placeholder="Jan Kowalski"
        />
        <span v-if="errors.name" class="error-text">{{ errors.name }}</span>
      </div>

      <div class="input-group">
        <label for="tel">Numer telefonu</label>
        <!-- <input
          v-model="form.tel"
          type="tel"
          id="tel"
          :class="{ 'input-error': errors.tel }"
          placeholder="+48 000 000 000"
        /> -->
        <span v-if="errors.tel" class="error-text">{{ errors.tel }}</span>
      </div>

      <div class="input-group">
        <label for="slices">Ile zjesz kawałków</label>
        <select
          v-model="form.slice"
          id="slices"
          :class="{ 'input-error': errors.slice }"
        >
          <option :value="null" disabled>Wybierz liczbę...</option>
          <option v-for="value in slicesOptions" :key="value" :value="value">
            {{ value }}
          </option>
        </select>
        <span v-if="errors.slice" class="error-text">{{ errors.slice }}</span>
      </div>

      <div class="input-group">
        <label for="paied">Metoda płatności</label>
        <select
          v-model="form.paied"
          id="paied"
          :class="{ 'input-error': errors.paied }"
        >
          <option value="" disabled>Wybierz płatność...</option>
          <option value="blik">Blik</option>
          <option value="cash">Gotówka</option>
        </select>
        <label>Ja będę zamawiał</label>
        <input type="checkbox" v-model="form.manager">

        {{ console.log(form.manager) }}
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
      form: {
        name: "",
        paied: "",
        slice: null,
        manager: false,
        isPaingPerson: false, 
      },
      slicesOptions: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
      errors: {},
    };
  },

  mounted() {
    console.log("Obecne ciasteczka:", document.cookie);
    const savedName = Cookies.get("user_name");
    if (savedName) {
      this.form.name = savedName;
    }
  },

  methods: {
    validateForm() {
      const isNameValid = !!this.form.name && this.form.name.length >= 3;
      const isSliceValid = !!this.form.slice;
      const isPaymentValid = !!this.form.paied;

      if (!isNameValid || !isSliceValid || !isPaymentValid) {
        alert("Przerósł cię formularz. Wypełnij cały");
        return false;
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
        Cookies.set("user_name", this.form.name, { expires: 100 });
        console.log("Ciasteczko zapisane!");
      }
    },
  },
};
</script>
<style scoped>
/* Kontener, który wyśrodkowuje kartkę */
.form-container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 20px;
  min-height: auto; /* Dostosuj w razie potrzeby */
}

/* "Nowy bloczek kelnera" */
.form-card {
  background: white;
  padding: 30px;
  border-radius: 8px; /* Lekkie zaokrąglenie */
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1); /* Subtelniejszy cień dla białego papieru */
  max-width: 400px;
  width: 100%;
  position: relative;
  font-family: "Courier New", Courier, monospace; /* Styl pisma maszynowego */
  overflow: visible;
}

/* Efekt postrzępienia u góry, jak przed wyrwaniem */
.form-card::before {
  content: "";
  position: absolute;
  top: -12px;
  left: 0;
  right: 0;
  height: 20px;
  background-image: radial-gradient(circle, transparent 40%, white 45%);
  background-size: 24px 24px;
  background-position: center;
  border-bottom: 2px solid #ffcccc; /* Delikatna czerwona linia */
}

/* Tytuł */
.form-title {
  text-align: center;
  margin-top: 10px;
  margin-bottom: 30px;
  color: #333;
  text-transform: uppercase;
  border-bottom: 1px solid #ccc;
  padding-bottom: 10px;
  font-size: 1.2rem;
}

/* Grupy wejściowe */
.input-group {
  margin-bottom: 25px;
  position: relative;
}

/* Etykiety */
.input-group label {
  display: block;
  font-weight: bold;
  color: #555;
  margin-bottom: 5px;
  text-transform: uppercase;
  font-size: 0.8rem;
}

/* Pola tekstowe - zamiast szarych pudełek, linie! */
.input-group input,
.input-group select {
  width: 100%;
  padding: 10px 5px;
  border: none;
  background-color: transparent;
  font-family: inherit; /* Używa Courier */
  font-size: 1rem;
  color: #333;
  border-bottom: 2px dashed #bbb; /* Przerywana linia jak w notesie */
  outline: none;
  transition: border-color 0.2s;
}

/* Stan skupienia (kliknięcia) */
.input-group input:focus,
.input-group select:focus {
  border-bottom-color: #e63946; /* Czerwona linia przy pisaniu */
}

/* Stylowanie selecta */
.input-group select {
  cursor: pointer;
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 16px;
}

/* Placeholder */
.input-group input::placeholder {
  color: #bbb;
  font-style: italic;
}

/* Przycisk - jak fioletowa etykieta */
.submit-btn {
  display: block;
  width: 100%;
  padding: 15px;
  margin-top: 40px;
  background-color: #6c5ce7; /* Twój fiolet */
  color: white;
  border: 1px dashed white; /* Dodaje surowości */
  border-radius: 6px;
  font-family: inherit;
  font-size: 1.1rem;
  font-weight: bold;
  text-transform: uppercase;
  cursor: pointer;
  box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.2);
  transition: transform 0.1s, box-shadow 0.1s;
  position: relative;
}

/* Efekt kliknięcia */
.submit-btn:active {
  transform: translateY(2px);
  box-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

/* Opcjonalnie: Postrzępiony efekt na krawędziach przycisku (bardziej zaawansowane) */
.submit-btn::before,
.submit-btn::after {
  content: "";
  position: absolute;
  top: 0;
  height: 100%;
  width: 10px;
}
</style>
