<template>
  <div>
    <div>
      <input
        type="search"
        v-model="searchQuery"
        placeholder="Search Laureate"
      />
      <button @click="search">Search</button>
    </div>
    <button @click="previous" :disabled="currentPage == 0">prev</button>
    <label>{{ currentPage + 1 }}/{{ pages }}</label>
    <button @click="next" :disabled="currentPage == pages - 1">next</button>
  </div>

  <table id="tableComponent">
    <caption>
      Nobel Prize Laureates
    </caption>
    <thead>
      <tr>
        <!-- loop through each value of the fields to get the table header -->
        <th v-for="field in fields" :key="field" @click="sortTable(field)">
          {{ field }}
        </th>
      </tr>
    </thead>
    <tbody>
      <!-- Loop through the list get the each student data -->
      <tr v-for="laureate in paginatedLaureates" :key="laureate">
        <td v-for="field in fields" :key="field">{{ laureate[field] }}</td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import { getAuth, onAuthStateChanged } from "firebase/auth";
import { useRouter } from "vue-router";
import { getDatabase, ref, onValue } from "firebase/database";

const auth = getAuth();
const router = useRouter();
const db = getDatabase();

export default {
  data() {
    return {
      laureates: [],
      searchQuery: "",
      fields: ["N0", "Surname", "Name", "Country", "Category", "Year"],
      currentPage: 0,
      pageItems: 20,
      sortedLaureates: [],
      asc: true,
      a: [],
    };
  },

  computed: {
    pages() {
      return Math.ceil(this.sortedLaureates.length / this.pageItems);
    },
    paginatedLaureates() {
      let start = this.currentPage * this.pageItems;
      let end = start + this.pageItems;
      return this.sortedLaureates.slice(start, end);
    },
  },

  methods: {
    next() {
      this.currentPage++;
    },
    previous() {
      this.currentPage--;
    },
    sortTable(field) {
      if (this.asc) {
        this.sortedLaureates.sort((x, y) => (x[field] > y[field] ? -1 : 1));
        this.asc = false;
      } else {
        this.sortedLaureates.sort((x, y) => (x[field] < y[field] ? -1 : 1));
        this.asc = true;
      }
    },

    search() {
      this.sortedLaureates = [];
      this.laureates.forEach((laureate) => {
        if (
          laureate.Name.toLowerCase().includes(this.searchQuery) ||
          laureate.Surname.toLowerCase().includes(this.searchQuery)
        ) {
          this.sortedLaureates.push(laureate);
        }
      });
    },
  },

  mounted() {
    this.sortedLaureates = this.laureates;
    const laureatesRef = ref(db, "laureates/");

    onAuthStateChanged(auth, (user) => {
      if (!user) {
        alert(
          "you must be logged in to view this. redirecting to the home page"
        );
        router.push("/");
      }
    });

    onValue(laureatesRef, (snapshot) => {
      let i = 0;
      const laureateData = snapshot.val();
      laureateData.forEach((laureate) => {
        i++;
        this.laureates.push({
          N0: i,
          Surname: laureate.surname,
          Name: laureate.firstname,
          Country: laureate.bornCountryCode,
          Category: laureate.prizes[0].category,
          Year: laureate.prizes[0].year,
        });
      });
    });
  },
};
</script>