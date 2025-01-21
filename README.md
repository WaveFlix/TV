<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SPORTWAVE Movies</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            background-color: #181818; /* Dark background */
            color: #f1f1f1; /* Light text */
            display: flex;
            flex-direction: column;
            min-height: 100vh; /* Ensure full page height */
            padding: 0;
            box-sizing: border-box;
        }

        /* Translucent Navbar Style */
        .navbar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            padding: 20px;
            background-color: rgba(0, 0, 0, 0.7); /* Translucent background */
            z-index: 100;
        }

        .logo {
            width: 150px;
            height: auto;
        }

        .search-bar {
            display: flex;
            align-items: center;
        }

        .search-bar input {
            padding: 10px;
            border-radius: 4px;
            border: 1px solid #444; /* Dark border */
            background-color: #333;
            color: #f1f1f1;
            width: 250px;
        }

        .search-bar button {
            padding: 10px 16px;
            background-color: #e50914; /* Red button */
            border: none;
            color: white;
            cursor: pointer;
            border-radius: 4px;
            margin-left: 10px;
        }

        .search-bar button:hover {
            background-color: #b00710;
        }

        /* Movie Container */
        .movie-container {
            margin-top: 150px; /* More space between navbar and categories */
            padding: 40px 20px;
            flex-grow: 1;
            background-color: #181818; /* Dark background */
            overflow: hidden;
        }

        .movie-category {
            margin-bottom: 40px;
        }

        .category-title {
            font-size: 30px;
            margin-bottom: 20px;
            color: #e50914; /* Red color */
            font-weight: bold;
        }

        .movie-row {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .movie-card {
            transition: transform 0.3s ease;
            border-radius: 12px;
            overflow: hidden;
            background-color: #333; /* Dark movie card */
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.7);
        }

        .movie-card img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        .movie-card h3 {
            text-align: center;
            margin-top: 10px;
            font-size: 18px;
            color: #f1f1f1; /* Light text */
            padding: 10px;
            background-color: #444;
        }

        .movie-card:hover {
            transform: scale(1.05);
        }

        /* Category Button Style */
        .category-container {
            display: flex;
            gap: 15px;
            overflow-x: auto; /* Allow horizontal scrolling */
            padding: 10px 0;
            margin-bottom: 30px;
        }

        .category-button {
            padding: 12px 25px;
            background-color: #444;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }

        .category-button:hover {
            background-color: #e50914; /* Red hover */
        }

        .category-button:active {
            background-color: #b00710; /* Dark red on click */
        }

        /* Hide scrollbar for better design */
        .movie-row::-webkit-scrollbar {
            display: none;
        }

        /* Responsive Design */
        @media screen and (max-width: 768px) {
            .search-bar input {
                width: 200px;
            }
        }

        /* Mobile */
        @media screen and (max-width: 480px) {
            .movie-card {
                width: 160px;
            }

            .search-bar input {
                width: 150px;
            }
        }
    </style>
</head>
<body>

    <!-- Navbar -->
    <div class="navbar">
        <img src="https://i.ibb.co/vj5yj2B/Wave.png" alt="SPORTWAVE Logo" class="logo">
        <div class="search-bar">
            <input type="text" id="search-input" placeholder="Search for a movie...">
            <button onclick="searchMovies()">Search</button>
        </div>
    </div>

    <!-- Main Content -->
    <div class="movie-container">

        <!-- Category Buttons (Genre Selection) -->
        <div class="category-container" id="category-container"></div>

        <!-- Movie Categories and Posters -->
        <div id="movie-container"></div>
    </div>

    <script>
        const TMDB_API_KEY = '34b1d9f567ce0d9a84994eda20ee4d01';
        const STREAM_API_URL = 'https://vidlink.pro/movie/';
        const TMDB_POPULAR_URL = `https://api.themoviedb.org/3/movie/popular?api_key=${TMDB_API_KEY}&language=en-US&page=1`;
        const TMDB_SEARCH_URL = `https://api.themoviedb.org/3/search/movie?api_key=${TMDB_API_KEY}&language=en-US&query=`;
        const TMDB_GENRES_URL = `https://api.themoviedb.org/3/genre/movie/list?api_key=${TMDB_API_KEY}&language=en-US`;

        const movieContainer = document.getElementById('movie-container');
        const categoryContainer = document.getElementById('category-container');

        // Fetch and display popular movies on page load
        async function fetchPopularMovies() {
            try {
                const response = await fetch(TMDB_POPULAR_URL);
                const data = await response.json();
                displayMovies(data.results);
            } catch (error) {
                console.error('Error fetching popular movies:', error);
            }
        }

        // Fetch and display categories
        async function fetchGenres() {
            try {
                const response = await fetch(TMDB_GENRES_URL);
                const data = await response.json();
                displayGenres(data.genres);
            } catch (error) {
                console.error('Error fetching genres:', error);
            }
        }

        // Display genres in a horizontal scrollable list
        function displayGenres(genres) {
            genres.forEach(genre => {
                const button = document.createElement('button');
                button.classList.add('category-button');
                button.innerText = genre.name;
                button.onclick = () => fetchMoviesByGenre(genre.id);
                categoryContainer.appendChild(button);
            });
        }

        // Fetch and display movies for a specific genre
        async function fetchMoviesByGenre(genreId) {
            const url = `https://api.themoviedb.org/3/discover/movie?api_key=${TMDB_API_KEY}&with_genres=${genreId}`;
            try {
                const response = await fetch(url);
                const data = await response.json();
                displayMovies(data.results);
            } catch (error) {
                console.error('Error fetching movies by genre:', error);
            }
        }

        // Search for movies
        async function searchMovies() {
            const query = document.getElementById('search-input').value.trim();
            if (!query) {
                alert('Please enter a movie title to search.');
                return;
            }

            try {
                const response = await fetch(`${TMDB_SEARCH_URL}${encodeURIComponent(query)}`);
                const data = await response.json();
                if (data.results.length > 0) {
                    displayMovies(data.results);
                } else {
                    movieContainer.innerHTML = `<p>No movies found for "${query}".</p>`;
                }
            } catch (error) {
                console.error('Error searching for movies:', error);
            }
        }

        // Display movies in the container
        function displayMovies(movies) {
            movieContainer.innerHTML = ''; // Clear previous results
            const movieCategory = document.createElement('div');
            movieCategory.classList.add('movie-category');
            movieCategory.innerHTML = `<div class="category-title">Popular Movies</div>`;

            const movieRow = document.createElement('div');
            movieRow.classList.add('movie-row');

            movies.forEach(movie => {
                const movieCard = document.createElement('div');
                movieCard.classList.add('movie-card');

                const posterPath = movie.poster_path 
                    ? `https://image.tmdb.org/t/p/w500${movie.poster_path}` 
                    : 'https://via.placeholder.com/220x330?text=No+Image';

                movieCard.innerHTML = `
                    <img src="${posterPath}" alt="${movie.title}">
                    <h3>${movie.title}</h3>
                `;
                movieCard.onclick = () => window.open(`${STREAM_API_URL}${movie.id}`, '_blank');
                movieRow.appendChild(movieCard);
            });

            movieCategory.appendChild(movieRow);
            movieContainer.appendChild(movieCategory);
        }

        // Load popular movies and genres on initial page load
        fetchPopularMovies();
        fetchGenres();
    </script>
</body>
</html>
