<template>
  <div class="app">
    <header>
      <div class="navbar">
        <img src="../assets/logo.png" alt="Marvel Comics Logo" />
        <div class="favorites" @click="showFavorites = !showFavorites">
          <span class="badge heart-badge">
            Added to Favorite
            <span class="count">{{ favoriteCount }}</span>
          </span>
        </div>
      </div>
    </header>
    <main>

      <div class="comics-container">
      
        <div v-for="comic in comics" :key="comic.id" class="comic-card">
      
          <h3 class="comic-title">{{ comic.title }}</h3>
          <p v-if="comic.description" class="comic-description">
              {{ comic.description }}
            </p>
          <div class="comic-image">
            <img
              v-if="comic.thumbnail"
              :src="comic.thumbnail"
              :alt="comic.title"
            />
            <div v-else class="no-image-placeholder">No Image</div>
          </div>
          <div class="comic-details">
            <p class="comic-creators">
              <strong>Creators:</strong> {{ comic.creators }}
            </p>
            <button class="favorite-button" @click="toggleFavorite(comic)">
              <span class="heart-icon">{{
                isFavorite(comic) ? "❤️" : "🤍"
              }} </span>
            </button>
          </div>
        </div>
      </div>
    </main>
    <div class="favorites-modal" v-if="showFavorites">
      <h3>Favorites</h3>
      <div v-if="favorites.length === 0">No favorites added yet.</div>
      <div class="favorites-list" v-else>
        <div v-for="comic in favorites" :key="comic.id" class="favorite-item">
          <img :src="comic.thumbnail" alt="Comic Thumbnail" />
          <div class="favorite-details">
            <h4>{{ comic.title }}</h4>

            <p class="creators">Creators: {{ getCreators(comic) }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import md5 from "md5";

export default {
  data() {
    return {
      comics: [],
      favorites: [],
      showFavorites: false,
    };
  },
  computed: {
    favoriteCount() {
      return this.favorites.length;
    },
  },
  mounted() {
    this.fetchComics();
  },
  methods: {
    toggleFavorite(comic) {
      if (this.isFavorite(comic)) {
        this.removeFavorite(comic);
      } else {
        this.addFavorite(comic);
      }
    },
    async fetchComics() {
      const publicKey = "3e31088285e26afb65893e508b7ef006";
      const privateKey = "b352790b938640bd2f9b4232304cb7d6b7d39a45";
      const timestamp = new Date().getTime().toString();
      const hash = md5(timestamp + privateKey + publicKey);

      const response = await axios.get(
        "https://gateway.marvel.com/v1/public/comics",
        {
          params: {
            ts: timestamp,
            apikey: publicKey,
            hash: hash,
            format: "comic",
            formatType: "comic",
            noVariants: true,
            orderBy: "-modified",
            limit: 30,
          },
        }
      );

      this.comics = response.data.data.results.map((comic) => {
        return {
          id: comic.id,
          title: comic.title,
          description: comic.description,
          thumbnail: comic.thumbnail.path + "." + comic.thumbnail.extension,
          creators: comic.creators.items
            .map((creator) => creator.name)
            .join(", "),
        };
      });
    },
    addToFavorites(comic) {
      if (this.isFavorite(comic)) {
        this.removeFavorite(comic);
      } else {
        this.addFavorite(comic);
      }
    },
    addFavorite(comic) {
      this.favorites.push(comic);
    },
    removeFavorite(comic) {
      const index = this.favorites.findIndex((item) => item.id === comic.id);
      if (index !== -1) {
        this.favorites.splice(index, 1);
      }
    },
    isFavorite(comic) {
      return this.favorites.some((item) => item.id === comic.id);
    },
    getCreators(comic) {
      return comic.creators;
    },
  },
};
</script>
<style>
body {
  background-color: #222;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

header {
  margin-bottom: 20px;
  background-color: red;
  padding: 10px;
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 999;
}

h1 {
  font-size: 24px;
  font-weight: bold;
}

.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.navbar img {
  width: 150px;
}

.favorites {
  display: flex;
  align-items: center;
  color: #fff;
  cursor: pointer;
}

.favorites i {
  margin-right: 5px;
}

.favorite-button {
  display: flex;
  align-items: center;
  padding: 5px;
  background-color: transparent;
  border: none;
  cursor: pointer;
}

.heart-icon {
  margin-right: 5px;
}

.favorite-count {
  font-weight: bold;
}
.count{

  margin-left: 10px;
}
main {
  margin-top: 80px;
  padding: 20px;
}

.comics-container {
  background-color: #222;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  grid-gap: 20px;
}

.comic-card {
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.comic-image img {
  width: 100%;
  height: auto;
}

.no-image-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 200px;
  background-color: #eee;
  color: #666;
}

.comic-details {
  padding: 10px;
  flex-grow: 1;
}

.comic-title {
  font-size: 15px;
  color: #ddd;
  font-weight: bold;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
.comic-description {
  color: #eee;
  margin-top: 10px;
  font-size: 14px;
  line-height: 1.3;
  max-height: 5.2em;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 5;
  -webkit-box-orient: vertical;
}
.comic-creators {
  margin-top: 10px;
  font-size: 14px;
  color: #666;
}

.favorite-btn {
  background: none;
  border: none;
  font-size: 1rem;
  color: #ff0000;
  cursor: pointer;
}

.favorites-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  padding: 20px;
  color: #fff;
}

.favorites-modal h3 {
  margin-top: 0;
}

.favorites-list {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.favorite-item {
  background-color: #222;
  padding: 10px;
  text-align: center;
}

.favorite-item img {
  max-width: 100%;
}

.favorite-item h4 {
  margin-top: 10px;
  margin-bottom: 5px;
  font-size: 14px;
  color: #fff;
}

.favorite-item .creators {
  margin: 0;
  font-size: 12px;
  color: #ccc;
}
.heart-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  color: rgb(253, 253, 253);
  font-size: 24px;
  border: none;
  padding: 2px;
}

h3{
  margin: 20px;
}

@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    align-items: center;
  }

  .favorites {
    margin-top: 1rem;
  }

  .comics-container {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  }

  .comic-card {
    height: 300px;
  }
}

@media (max-width: 480px) {
  .comics-container {
    grid-template-columns: 1fr;
  }
}
</style>
