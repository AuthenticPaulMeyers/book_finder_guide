# Student Guide: Building a Book Finder Web Application

In this project, you will build an interactive **Book Finder** web application using HTML, CSS, and vanilla JavaScript. 

Use the provided link to as reference to what you need to accomplish: 
LINK: https://authenticpaulmeyers.github.io/book_finder_template/

This project will help you put key intermediate JavaScript concepts into practice, including DOM manipulation, asynchronous programming (`fetch` & `Promise`s), error handling, API interaction, JSON parsing, and managing local storage.

---

## Project Overview & Requirements

Your goal is to build a web application that connects to the free [Gutendex API](https://gutendex.com/), a public REST API providing access to Project Gutenberg books.

### Core Features
1. **Browse Books:** Display a grid/list of books fetched from the Gutendex API upon loading.
2. **Search & Filter:** Allow users to search for books by keyword and filter or sort by topic/genre.
3. **Book Details Modal:** Clicking on a book card opens a popup modal with extra details (authors, subjects, download counts, and languages).
4. **Favorites System:** Users can add books to a "Favorites" list or remove them. Favorites should persist across page refreshes using `localStorage`.

---

## Concepts Covered

- **DOM Manipulation:** Dynamically creating, styling, and inserting HTML elements.
- **Fetch API & HTTP Methods:** Sending `GET` requests to retrieve data from an external server.
- **Promises & `async`/`await`:** Handling asynchronous data streams cleanly.
- **Error Handling:** Using `try...catch` blocks and checking HTTP status codes (`response.ok`).
- **JSON:** Parsing incoming HTTP responses into JavaScript objects.
- **Browser Storage:** Storing and retrieving data using `localStorage`.

---

## Project Structure

Organize your project files as follows:

```
book-finder/
> index.html
> styles.css
> app.js

```

---

## Step-by-Step Implementation Guide

### Step 1: Setting Up the HTML Skeleton (`index.html`)

Create an intuitive layout containing:
1. A **header** with a title and a toggle/button to view favorite books.
2. A **search & filter controls bar**.
3. A **modal overlay** hidden by default for displaying book details.

```html
// Note: This is just a sample skeleton make sure you integrate other elements to achieve the given web document reference.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Book Finder App</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>

  <header>
    <h1>Book Finder</h1>
    <button id="view-favorites-btn">View Favorites (<span id="fav-count">0</span>)</button>
  </header>

  <main>
    <!-- Controls Section -->
    <section class="controls">
      <input type="text" id="search-input" placeholder="Search by title or author..." />
      <select id="topic-select">
        <option value="">All Topics</option>
        <option value="fiction">Fiction</option>
        <option value="mystery">Mystery</option>
        <option value="history">History</option>
        <option value="science">Science</option>
      </select>
      <button id="search-btn">Search</button>
    </section>

    <!-- Status Messages -->
    <div id="status-message" class="hidden"></div>

    <!-- Books Container -->
    <section id="books-container" class="books-grid"></section>
  </main>

  <!-- Modal for Book Details -->
  <div id="book-modal" class="modal hidden">
    <div class="modal-content">
      <span id="close-modal" class="close-btn">&times;</span>
      <div id="modal-body"></div>
    </div>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

---

### Step 2: Styling the Application (`styles.css`)

Add CSS styles to lay out the grid and handle modal visibility.

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

body {
  padding: 20px;
  background-color: #f4f6f8;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.controls input, .controls select, .controls button {
  padding: 10px;
  font-size: 1rem;
}

.books-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
}

.book-card {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.book-card img {
  width: 100%;
  height: 250px;
  object-fit: cover;
  border-radius: 4px;
}

/* Modal Styling */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 1.5rem;
  cursor: pointer;
}

.hidden {
  display: none !important;
}
```

---

### Step 3: Fetching Data from the API (`app.js`)

The Gutendex API endpoint base URL is `https://gutendex.com/books`.

```javascript
const API_BASE_URL = "https://gutendex.com/books/?page=2";

function fetchData(URL){
      fetch(URL).then(response => {
                  if(!response.ok){ 
                        throw new Error(`Error: ${response.status}`);  
                  }
                  return response.json();  

            }).then(data => {

            // Data is an object with a property called results, which is an array of books
            const books = data.results;
            books.forEach(book => {
              // placeholders
                  console.log(`Title: ${book.title}`);
                  console.log(`Author: ${book.authors[0].name}`);
                  console.log("--------------------------------------------------");
            
            });
      }).catch((error) => {
            console.log(`This is an error: ${error}`);
});
};

// Run the function
fetchData(API_BASE_URL);
      
---

### Step 4: Rendering Books in the DOM

Create HTML elements dynamically to render book cards on the page.

```javascript
const booksContainer = document.getElementById('books-container');

function renderBooks(books) {
  booksContainer.innerHTML = ''; // Clear previous content

  books.forEach(book => {
    const card = document.createElement('div');
    card.classList.add('book-card');

    // Extract cover image or fall back to a placeholder
    const coverUrl = book.formats['image/jpeg'] || 'https://via.placeholder.com/150?text=No+Cover';
    const authors = book.authors.map(a => a.name).join(', ') || 'Unknown Author';

    card.innerHTML = `
      <img src="${coverUrl}" alt="${book.title}" />
      <h3>${book.title}</h3>
      <p><strong>Author:</strong> ${authors}</p>
      <button class="details-btn" data-id="${book.id}">View Details</button>
      <button class="fav-btn" data-id="${book.id}">
        ${isFavorite(book.id) ? '❤️ Remove Favorite' : '🤍 Add Favorite'}
      </button>
    `;

    // Event listener for opening details modal
    card.querySelector('.details-btn').addEventListener('click', () => {
      openBookModal(book);
    });

    // Event listener for toggling favorite status
    card.querySelector('.fav-btn').addEventListener('click', (e) => {
      toggleFavorite(book);
      renderBooks(books); // Refresh card state
    });

    booksContainer.appendChild(card);
  });
}
```

---

### Step 5: Implementing Details Popup Modal

Display comprehensive book details inside a modal when clicked.

```javascript
const modal = document.getElementById('book-modal');
const modalBody = document.getElementById('modal-body');
const closeModalBtn = document.getElementById('close-modal');

function openBookModal(book) {
  const authors = book.authors.map(a => `${a.name} (${a.birth_year || '?'} - ${a.death_year || '?'})`).join(', ');
  const subjects = book.subjects.slice(0, 5).join(', ') || 'None listed';

  modalBody.innerHTML = `
    <h2>${book.title}</h2>
    <p><strong>Authors:</strong> ${authors}</p>
    <p><strong>Subjects:</strong> ${subjects}</p>
    <p><strong>Languages:</strong> ${book.languages.join(', ').toUpperCase()}</p>
    <p><strong>Total Downloads:</strong> ${book.download_count}</p>
  `;

  modal.classList.remove('hidden');
}

closeModalBtn.addEventListener('click', () => {
  modal.classList.add('hidden');
});

// Close modal when clicking outside content box
window.addEventListener('click', (e) => {
  if (e.target === modal) {
    modal.classList.add('hidden');
  }
});
```

---

### Step 6: Managing Favorites with `localStorage`

Store user's favorited books inside `localStorage` so data remains saved after refreshing the page.

```javascript
let favorites = JSON.parse(localStorage.getItem('favorites')) || [];

function isFavorite(bookId) {
  return favorites.some(b => b.id === bookId);
}

function toggleFavorite(book) {
  if (isFavorite(book.id)) {
    favorites = favorites.filter(b => b.id !== book.id);
  } else {
    favorites.push(book);
  }
  
  localStorage.setItem('favorites', JSON.stringify(favorites));
  updateFavCount();
}

function updateFavCount() {
  document.getElementById('fav-count').textContent = favorites.length;
}
```

---

### Step 7: Connecting Search, Filtering & Event Handling

Wire all user inputs to event listeners to handle user interactions dynamically.

```javascript
// Initialization
document.addEventListener('DOMContentLoaded', async () => {
  updateFavCount();
  const books = await fetchBooks();
  renderBooks(books);
});

// Search & Filter Action
document.getElementById('search-btn').addEventListener('click', async () => {
  const query = document.getElementById('search-input').value.trim();
  const topic = document.getElementById('topic-select').value;
  
  const books = await fetchBooks(query, topic);
  renderBooks(books);
});

// View Favorites Toggle
let showingFavorites = false;
document.getElementById('view-favorites-btn').addEventListener('click', async (e) => {
  showingFavorites = !showingFavorites;
  
  if (showingFavorites) {
    e.target.firstChild.textContent = 'View All Books ';
    renderBooks(favorites);
  } else {
    e.target.firstChild.textContent = 'View Favorites ';
    const books = await fetchBooks();
    renderBooks(books);
  }
});
```

---

## Challenges & Next Steps for Students

Try adding these extra enhancements to make your application stand out:

1. **Pagination:** Gutendex provides `next` and `previous` page URLs in its JSON response. Implement Next/Previous pagination controls!
2. **Debounce Search Input:** Trigger search requests automatically as users type into the input field, using a debounce function to avoid excessive API requests.
3. **Loading and Error messages** - add state appropriately during network delays or bad network connections.

---

## Testing Checklist

Before submitting your project, verify:
- [ ] Books load automatically on initial page visit.
- [ ] Searching by keyword filters results properly.
- [ ] Selecting a topic filters books by subject correctly.
- [ ] Clicking a book opens the details modal with author/subject info.
- [ ] Adding/removing favorites updates the favorite counter.
- [ ] Favorited books persist after refreshing your browser.
