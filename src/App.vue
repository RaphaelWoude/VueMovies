<template>
  <div class="container py-4">
    <div class="row">
      <div class="col-6">
        <h1>Search</h1>
        <div class="form-group my-4">
          <input
            type="text"
            class="form-control"
            v-model="search"
            @input="getResults()"
          />
        </div>
      </div>
      <div class="col-6">
        <h1>Results</h1>
        <div v-for="result in results" :key="result" class="card flex-row flex-wrap">
          <div class="card-header border-0">
            <img :src="result.Poster" alt="" />
          </div>
          <div class="card-block p-4">
            <h4 class="card-title">{{ result.Title }}</h4>
            <h5 class="card-subtitle" v-html="getRating(result.imdbID)"></h5>
            <p class="card-text">{{ result.Year }}</p>
          </div>
          <div class="card-footer w-100">
            <div class="row">
              <div class="col-9">
                <div class="btn-group" role="group" aria-label="Basic example">
                  <button type="button" class="btn btn-secondary" @click="addHasSeen($event, result.imdbID)" :disabled="isHasSeen(result.imdbID)">Al gezien</button>
                  <button type="button" class="btn btn-secondary" @click="addWillSee($event, result.imdbID)" :disabled="isWillSee(result.imdbID)">Wil zien</button>
                </div>
              </div>
              <div class="col-3">
                <input type="number" class="form-control" name="rating" min=1 max=5 v-model="rating" @input="addRating(result.imdbID)" placeholder="Rating">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data: function () {
    return {
      search: "",
      rating: "",
      results: [],
    };
  },
  methods: {
    async getResults() {
      this.results = [];

      await fetch(
        `http://www.omdbapi.com/?i=tt3896198&apikey=186be766&s=${this.search}`
      ).then(async (response) => {
        if (response.status !== 200) {
          return;
        }

        const body = await response.json();

        if (!body.Search) {
          return;
        }
        for (const entry of body.Search) {
          this.results.push(entry);
        }
      });
    },
    addHasSeen($event, imdbID) {
      const storage = this.getFromStorage("hasSeen");

      // Create storage with imdbID. No need to check as no storage exist yet.
      if (!storage) {
        this.setToStorage("hasSeen", imdbID);
        return;
      }

      for (const entry of storage) {
        if (entry.imdbID === imdbID) {
          return;
        }
      }

      this.setToStorage("hasSeen", imdbID);
      $event.target.disabled = true;
    },
    addRating(imdbID) {
      const storage = this.getFromStorage("ratings");

      // Create storage with imdbID. No need to check as no storage exist yet.
      if (!storage) {
        this.setToStorage("ratings", {
          rating: this.rating,
          imdbID
        });
        return;
      }

      for (const entry of storage) {
        if (entry.imdbID === imdbID) {
          return;
        }
      }

      this.setToStorage("ratings", {
        rating: this.rating,
        imdbID
      });
    },
    getRating(imdbID) {
      const storage = this.getFromStorage("ratings");

      if (!storage) {
        return 'No Rating';
      }
      
      for (const entry of storage) {
        if (entry.imdbID.imdbID === imdbID) {
          return 'Rating: ' + entry.imdbID.rating;
        }
      }

      return 'No Rating';
    },
    isHasSeen(imdbID) {
      const storage = this.getFromStorage("hasSeen");

      if (!storage) {
        return false;
      }
  
      for (const entry of storage) {
        if (entry.imdbID === imdbID) {
          return true;
        }
      }

      return false;
    },
    addWillSee($event, imdbID) {
      const storage = this.getFromStorage("willSee");

      // Create storage with imdbID. No need to check as no storage exist yet.
      if (!storage) {
        this.setToStorage("willSee", imdbID);
        return;
      }

      for (const entry of storage) {
        if (entry.imdbID === imdbID) {
          return;
        }
      }

      this.setToStorage("willSee", imdbID);
      $event.target.disabled = true;
    },
    isWillSee(imdbID) {
      const storage = this.getFromStorage("hasSeen");

      if (!storage) {
        return false;
      }
  
      for (const entry of storage) {
        if (entry.imdbID === imdbID) {
          return true;
        }
      }

      return false;
    },
    setToStorage(name, imdbID) {
      let items = JSON.parse(localStorage.getItem(name));
      if (items == null) {
        items = [];
      }

      const item = { imdbID };
      if (items.includes(item)) {
        return;
      }

      items.push(item);
      localStorage.setItem(name, JSON.stringify(items));
    },
    getFromStorage(name) {
      let items = JSON.parse(localStorage.getItem(name));
      return items;
    }
  },
};
</script>

<style scoped>
  img {
    width: 300px;
    height: 445px;
  }
</style>
