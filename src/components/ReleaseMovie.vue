<template>
    <div class="container">
        <h1 class="title">Released Movies in 2023</h1>
        <ul class="movie-list">
            <li v-for="movie in displayedMovies" :key="movie.id" class="movie-item">
                <div class="movie-card">
                    <img :src="getBannerImageUrl(movie)" alt="Movie Banner" class="banner-image" @click="showMoviePopup(movie)" />
                    <div class="movie-details" @click="showMoviePopup(movie)">
                        <h2 class="movie-title">{{ movie.title }}</h2>
                        <p class="movie-overview">{{ movie.overview }}</p>
                        <div class="movie-rating">
                            <span class="rating-label">Rating:</span>
                            <span class="rating-value">{{ movie.vote_average }}</span>
                        </div>
                    </div>
                </div>
            </li>
        </ul>
        <div v-if="selectedMovie" class="popup">
            <div class="popup-content">
                <div class="popup-header">
                    <img :src="getBannerImageUrl(selectedMovie)" alt="Movie Banner" class="popup-banner-image" />
                    <div class="popup-details">
                        <h2 class="popup-title">{{ selectedMovie.title }}</h2>
                        <p class="popup-genre">{{ selectedMovie.genre }}</p>
                        <div class="popup-rating">
                            <span class="rating-label">Rating:</span>
                            <span class="rating-value">{{ selectedMovie.vote_average }}</span>
                        </div>
                    </div>
                </div>
                <p class="popup-overview">{{ selectedMovie.overview }}</p>
                <button class="close-button" @click="closeMoviePopup">Close</button>
            </div>
        </div>
        <div v-if="showPagination" class="pagination">
            <button @click="goToPage(1)" :disabled="currentPage === 1">First</button>
            <button @click="previousPage" :disabled="currentPage === 1">Previous</button>
            <button v-for="page in visiblePages" :key="page" @click="goToPage(page)" :class="{ active: page === currentPage }">{{ page }}</button>
            <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
            <button @click="goToPage(totalPages)" :disabled="currentPage === totalPages">Last</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            movies: [],
            selectedMovie: null,
            currentPage: 1,
            moviesPerPage: 5,
            visiblePages: [],
        };
    },
    computed: {
        displayedMovies() {
            const startIndex = (this.currentPage - 1) * this.moviesPerPage;
            const endIndex = startIndex + this.moviesPerPage;
            return this.movies.slice(startIndex, endIndex);
        },
        totalPages() {
            return Math.ceil(this.movies.length / this.moviesPerPage);
        },
        showPagination() {
            return this.movies.length > this.moviesPerPage;
        },
    },
    mounted() {
        this.fetchMovies();
    },
    methods: {
        fetchMovies() {
            axios
                .get('https://api.themoviedb.org/3/movie/now_playing', {
                    params: {
                        api_key: '3c1b4b15413919726268789066e9700c',
                    },
                })
                .then((response) => {
                    this.movies = response.data.results;
                    this.updateVisiblePages();
                })
                .catch((error) => {
                    console.error(error);
                });
        },
        getBannerImageUrl(movie) {
            const baseUrl = 'https://image.tmdb.org/t/p/original';
            return baseUrl + movie.backdrop_path;
        },
        showMoviePopup(movie) {
            this.selectedMovie = movie;
        },
        closeMoviePopup() {
            this.selectedMovie = null;
        },
        previousPage() {
            if (this.currentPage > 1) {
                this.currentPage--;
                this.updateVisiblePages();
            }
        },
        nextPage() {
            if (this.currentPage < this.totalPages) {
                this.currentPage++;
                this.updateVisiblePages();
            }
        },
        goToPage(page) {
            if (page >= 1 && page <= this.totalPages) {
                this.currentPage = page;
                this.updateVisiblePages();
            }
        },
        updateVisiblePages() {
            const maxVisiblePages = 5;
            const halfVisiblePages = Math.floor(maxVisiblePages / 2);
            let startPage = Math.max(1, this.currentPage - halfVisiblePages);
            let endPage = Math.min(startPage + maxVisiblePages - 1, this.totalPages);

            if (endPage - startPage + 1 < maxVisiblePages) {
                startPage = Math.max(1, endPage - maxVisiblePages + 1);
            }

            this.visiblePages = Array.from({ length: endPage - startPage + 1 }, (_, i) => startPage + i);
        },
    },
};
</script>

<style scoped>
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.title {
    color: #333;
    font-size: 32px;
    margin-bottom: 20px;
}

.movie-list {
    list-style: none;
    padding: 0;
}

.movie-item {
    margin-bottom: 30px;
}

.movie-card {
    display: flex;
    align-items: center;
    background-color: #f5f5f5;
    border-radius: 10px;
    padding: 20px;
    cursor: pointer;
}

.banner-image {
    width: 200px;
    height: auto;
    margin-right: 20px;
    border-radius: 5px;
}

.movie-details {
    flex: 1;
}

.movie-title {
    font-size: 24px;
    margin-bottom: 10px;
}

.movie-overview {
    font-size: 16px;
    margin-bottom: 10px;
}

.movie-rating {
    display: flex;
    align-items: center;
}

.rating-label {
    font-size: 16px;
    margin-right: 5px;
}

.rating-value {
    font-size: 16px;
    font-weight: bold;
}

.popup {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
}

.popup-content {
    background-color: #fff;
    padding: 20px;
    border-radius: 5px;
    max-width: 600px;
    text-align: center;
}

.popup-header {
    display: flex;
    align-items: center;
    margin-bottom: 20px;
}

.popup-banner-image {
    width: 300px;
    height: auto;
    margin-right: 20px;
    border-radius: 5px;
}

.popup-details {
    flex: 1;
    text-align: left;
}

.popup-title {
    font-size: 24px;
    margin-bottom: 10px;
}

.popup-genre {
    font-size: 16px;
    margin-bottom: 10px;
}

.popup-rating {
    display: flex;
    align-items: center;
}

.popup-overview {
    font-size: 16px;
    margin-bottom: 10px;
}

.close-button {
    background-color: #333;
    color: #fff;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.pagination {
    margin-top: 20px;
}

.pagination button {
    margin-right: 10px;
}

.pagination button.active {
    background-color: #333;
    color: #fff;
}
</style>
