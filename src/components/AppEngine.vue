<template>
  <div id="App">
    <div class="container">
      <form class="searchInput">
        <input type="text" v-model="searchQuery" placeholder="Wyszukaj..."/>
      </form>
      <p></p>
      <Form ref="form" @submit="onSubmit" class="newItemform">
        <Field name="f_title" type="text" v-model="newItem.title" placeholder="Tytuł:" :rules="required"/>
        <ErrorMessage name="f_title" class="error"/>
        <p></p>
          <Field name="f_type" as="select" v-model="newItem.type" :rules="required">
            <option value="" selected disabled hidden>Wybierz typ:</option>
            <option value="drzewny">drzewny</option>
            <option value="na ziemny">na ziemny</option>
          </Field>
        <ErrorMessage name="f_type" class="error"/>
        <p></p>
        <Field name="f_price" type="number" placeholder="Cena:" v-model="newItem.price" :rules="validatePrice"/>
        <ErrorMessage name="f_price" class="error"/>
        <p></p>
        <Field name="f_animals" type="text" placeholder="Zwierzęta:" v-model="newItem.animals" :rules="required"/>
        <ErrorMessage name="f_animals" class="error"/>
        <p></p>
        <Field name="f_category" type="text" placeholder="Nowa kategoria:" v-model="newItem.category"/>
        <p></p>
        <button type="submit" class="addButton" prevent>Dodaj</button>
      </Form>
      <p></p>
      <div class="sortingOptions">
        <div class="waluta">Waluta:</div>
        <div class="currencybuttons">
          <button type="button" @click="backtoPLN">PLN</button>
          <button type="button" @click="currentCurrency = 'EUR'">EUR</button>
          <button type="button" @click="currentCurrency = 'USD'">USD</button>
          <button type="button" @click="currentCurrency = 'CAD'">CAD</button>
        </div>
      </div>
      <div class="sortbuttons">
        <button @click="sortByPriceAsc">Sortuj według ceny rosnąco</button>
        <button @click="sortByPriceDesc">Sortuj według ceny malejąco</button>
      </div> <p></p>
      <ul v-if="!searchQuery">
        <appItems v-for="item in items" :item="item" :current-currency="currentCurrency" :key="item.id" v-on:delete-item="deleteItem"/>
      </ul>
      <ul>
        <searchItems  v-for="item in filteredItems" :item="item" :current-currency="currentCurrency" :key="item.id" v-on:delete-item="deleteItem"/>
      </ul>
    </div>
  </div>
</template>
  
  <script>
  import appItems from './appItems.vue'
  import searchItems from './searchItems.vue'
  import { Form, Field, ErrorMessage } from 'vee-validate';

  export default {
    created() {
    // eslint-disable-next-line
    const API_KEY = 'MZHR2Fnh3jwyZPm32q6prn5z3N7l2Kc8CX177Xmm';
    const url = `https://api.freecurrencyapi.com/v1/latest?apikey=MZHR2Fnh3jwyZPm32q6prn5z3N7l2Kc8CX177Xmm&currencies=EUR%2CUSD%2CCAD&base_currency=PLN`;

    fetch(url)
        .then(response => response.json())
        .then(data => {
        this.currencies = data.data;
        console.log(this.currencies)
      })
        .catch(error => {
        console.error(error);
      });
    },
    components: {
      appItems,
      searchItems,
      Form,
      Field,
      ErrorMessage
    },
    data() {
      return {
        currentCurrency: 'PLN',
        newItem: {
          title: '',
          type: '',
          price: '',
          animals: '',
          id: '',
          category: '',
          startingPricePLN: 0,
        },
        searchQuery: '',
        items: [
          {title: 'Domek 1', type: 'drzewny', price: '100.99', animals: 'ptaki, wiewiórki', id: '1231234', category: 'domki dla zwierząt', startingPricePLN: 100.99 },
          {title: 'Domek 2', type: 'na ziemny', price: '200.84', animals: 'kot, pies', id: '53456', category: 'domki dla zwierząt', startingPricePLN: 200.84 }
        ],
        currencies: {}
      }
    },
      methods: {
            onSubmit() {
              console.log(this.$refs.form.invalid)
              if (this.$refs.form.invalid) {
                // 
              } else {
                this.addItem()
              }
            },
            required(value) {
              if (!value || value === '') {
                return 'To pole jest wymagane!';
              }
              return true;
            },
            validatePrice(value) {
              const regex = /^\d+(\.\d{1,2})?$/
              if (!regex.test(value)) {
                return 'Cena nie jest poprawna!'
              }
              if (value < 0) {
                return 'Cena nie może być ujemna!'
              }
              return true
            },
            addItem() {
              this.newItem.startingPricePLN = this.newItem.price;
              this.items.push(this.newItem);
              this.newItem = {
                title: '',
                type: '',
                price: '',
                animals: '',
                id: '',
                category: ''
              }
            },
            deleteItem(id) {
              const index = this.items.findIndex(item => item.id === id);
              this.items.splice(index, 1);
            },
            sortByPriceAsc() {
              this.items.sort((a, b) => a.price - b.price);
              console.log(this.items)
            },
            sortByPriceDesc() {
              this.items.sort((a, b) => b.price - a.price);
              console.log(this.items)
            },
            currencyChange() {
              const targetCurrency = this.currentCurrency;
              const conversionRate = this.currencies[targetCurrency];

              if (isNaN(conversionRate)) {
                console.error(`Nieprawidłowa konwersja dla ${targetCurrency}: ${conversionRate}`);
                return;
              }
              this.items.forEach((item) => {
                item.price = item.startingPricePLN * conversionRate;
                item.price = item.price.toFixed(2);
              });
            },
            backtoPLN() {
              this.items.forEach((item) => {
                item.price = item.startingPricePLN
              });
              this.currentCurrency = 'PLN'
            }
          },
        computed: {
          filteredItems() {
              if (!this.searchQuery) {
                  return [];
                }
        
                return this.items.filter(item => {
                return Object.values(item).some(value => {
                return value.toString().toLowerCase().includes(this.searchQuery.toLowerCase());
              });
            });
          }
        },
        watch: {
          currentCurrency: function () {
            this.currencyChange();
          }
        },
    }
  </script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@100&display=swap');

body {
    font-weight:bold;
    font-family: 'Roboto', sans-serif;
    margin:0;
    padding:0;
    background: rgb(36, 36, 36);
    background-repeat: no-repeat;
    min-height: 100vh;
}
.container {
  width:30vw;
  display:block;
  height:fit-content;
  background-color:rgb(22, 22, 22);
  padding:20px;
  position:absolute;
  margin:auto;
  top:0; right:0; bottom:0; left:0;
  border-radius: 2%;
  -webkit-box-shadow: 8px 8px 24px -8px rgba(0, 0, 0, 1);
  -moz-box-shadow: 8px 8px 24px -8px rgba(0, 0, 0, 1);
  box-shadow: 8px 8px 24px -8px rgba(0, 0, 0, 1);
}
@media only screen and (max-width: 930px) {
  .container {
    width: 85vw;
    top:30px;
  }
}
@media only screen and (max-width: 325px) {
  .container {
    width: 85vw;
    margin:0px;
    top:30px;
  }
}
input::placeholder {
  color:#000;
}
.searchInput {
  margin-bottom:20px;
}
.searchInput input{
  -webkit-box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  -moz-box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  width:98%;
  height:25px;
  padding:5px;
  border:1px solid rgb(190, 190, 190);
  border-radius:5px;
  background-color:rgb(190, 190, 190);
}
.newItemform input {
  -webkit-box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  -moz-box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  width:98%;
  height:25px;
  padding:5px;
  border:1px solid rgb(190, 190, 190);
  border-radius:5px;
  background-color:rgb(190, 190, 190);
  z-index: 1000;
}
  select {
    width:100%;
    height:35px;
    padding:5px;
    border:3px solid rgb(190, 190, 190);
    border-radius:5px;
    background-color:rgb(190, 190, 190);
    -webkit-box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
    -moz-box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
    box-shadow: 3px 3px 0px 0px rgb(14, 13, 13);
  }
  ul {
    margin:0px;
    padding:0px;
  }
  .error {
    display:block;
    margin:auto;
    font-weight:bolder;
    margin-top:-1px;
    text-align:center;
    width:97%;
    height:25px;
    border-bottom-right-radius: 5px;
    border-bottom-left-radius: 5px;
    text-align:center;
    padding:10px 5px 5px 5px;
    overflow:hidden;
    color:white;
    background-color:red;
    z-index:998;
  }
  .addButton {
    width:100%;
    border:1px solid rgb(103, 45, 240);
    background-color: rgb(103, 45, 240);
    height:fit-content;
    font-size:25px;
    color:white;
    padding:3px;
    border-radius:5px;
    -webkit-box-shadow: 3px 3px 0px 0px rgb(87, 38, 202);
    -moz-box-shadow: 3px 3px 0px 0px rgb(87, 38, 202);
    box-shadow: 3px 3px 0px 0px rgb(87, 38, 202);
    transition:0.1s;
  }
  .addButton:hover {
    margin: 3px 0 0 3px;
    padding:3px;
    box-shadow:none
  }
  .sortingOptions {
    display:inline-block;
    height:35px;
    margin-bottom:5px;
  }
  .waluta {
    display:block;
    font-size:25px;
    float:left;
    color:white;
  }
  .currencybuttons {
    display:inline-block;
  }
  .currencybuttons button {
    color:white;
    border:1px solid rgb(19, 180, 54);
    border-radius:5px;
    background-color: rgb(19, 180, 54);
    height:25px;
    margin:0 4px 4px 4px;
    -webkit-box-shadow: 3px 3px 0px 0px rgb(14, 175, 49);
    -moz-box-shadow: 3px 3px 0px 0px rgb(14, 175, 49);
    box-shadow: 3px 3px 0px 0px rgb(14, 175, 49);
    transition:0.1s;
  }
  .currencybuttons button:hover {
    margin:4px 4px 4px 8px;
    padding:3px;
    box-shadow:none
  }
  .currencybuttons button:focus{
    margin:4px 4px 4px 8px;
    padding:3px;
    box-shadow:none
  }
  .sortbuttons {
    width: 100%;
    height: fit-content;
    padding-bottom:10px;
    border-bottom:3px solid rgba(161, 161, 161, 1);
  }
  .sortbuttons button {
    width:100%;
    color:white;
    border:1px solid rgb(241, 82, 19);
    border-radius:5px;
    background-color:rgb(241, 82, 19);
    padding:6px;
    margin:3px 0 3px 0;
    -webkit-box-shadow: 3px 3px 0px 0px rgb(179, 62, 15);
    -moz-box-shadow: 3px 3px 0px 0px rgb(179, 62, 15);
    box-shadow: 3px 3px 0px 0px rgb(179, 62, 15);
    transition:0.1s;
  }
  .sortbuttons button:hover {
    padding:3px;
    box-shadow:none
  }
</style>