<template>
  <div>
    <div>
      <input class="inputText" v-model="artistName" placeholder="Enter the full name of the Musician" >
      <br><br>
      <button @click="birthdaySearch(artistName)">search</button>
      <button @click="clear">clear</button>
      <div v-if="errors.length">
          <h5 class="errorMessage" v-for="error in errors" :key="error.id">{{ error }}</h5>
      </div>
      <p>Musician name: {{ artistName }}</p>
      <p>Musician birthdate: {{ artistBirthday }}</p>
      <p>Musician hebrew birthdate: {{hebrewDate}}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      artistBirthday: "",
      artistName: "",
      hebrewDate: "",
      errors: []
    }
  },
  created() {},
  computed: {},
  methods: {
    birthdaySearch(name) {
      if(this.artistName === ""){
        return this.errors.push("Please enter artist full name")
      }
      axios.get(`http://musicbrainz.org/ws/2/artist/?query=name:` + name + `&fmt=json`)
          .then(response => {
            this.artistBirthday = response.data.artists[0]["life-span"]["begin"]
            if (this.artistBirthday === undefined) {
              this.errors.push('No Musician was found');
            }
            this.dateConverter(this.artistBirthday);
            this.errors = [];
          })
          .catch(e => {
            this.errors.push(e)
          })
    },
    dateConverter(dateToConvert) {
      let date = new Date(dateToConvert);
      if(!date.getDate()){
        return this.hebrewDate = "";
      }
      axios.post("https://www.hebcal.com/converter?cfg=json&gy=" + date.getFullYear() +
          "&gm=" + ((date.getMonth() + 1)) +
          "&gd=" + (date.getDate() > 9 ? '' : '0') + date.getDate() + "&g2h=1"
      )
          .then(response => {
            console.log("date response", response);
            this.hebrewDate = response.data.hebrew;
            this.errors = [];
          })
          .catch(e => {
            this.errors.push(e)
          })
    },
    clear() {
      this.artistName = "";
      this.artistBirthday = "";
      this.hebrewDate = "";
      this.errors = [];
    }
  }
}
</script>

<style scoped>
.inputText{
  text-align: center;
  width: 80%;
  height: 40px;
  font-size: 30px;
}

.errorMessage{
  color: #FF0000 ;
}
</style>
