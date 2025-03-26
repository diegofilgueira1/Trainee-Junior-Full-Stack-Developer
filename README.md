# Trainee-Junior-Full-Stack-Developer

# Backend Setup (Bun with Express, Axios, and JSDOM)

# Step 1: Initialize Bun Project
bun init amazon-scraper

# Step 2: Install Required Dependencies
bun add express axios jsdom

# Step 3: Create Backend Script (src/server.js)

import express from 'express';
import axios from 'axios';
import { JSDOM } from 'jsdom';

const app = express();
const PORT = process.env.PORT || 3000;

// Scraping logic
async function scrapeAmazon(keyword) {
  try {
    // Fetch Amazon search results page for the given keyword
    const response = await axios.get(`https://www.amazon.com/s?k=${keyword}`);
    const html = response.data;
    
    // Parse HTML using JSDOM
    const dom = new JSDOM(html);
    const document = dom.window.document;

    // Extract product listings
    const productElements = document.querySelectorAll('.s-main-slot .s-result-item');

    const products = [];

    productElements.forEach((product) => {
      const title = product.querySelector('h2')?.textContent || 'No Title';
      const rating = product.querySelector('.a-icon-alt')?.textContent || 'No Rating';
      const reviews = product.querySelector('.a-size-base')?.textContent || 'No Reviews';
      const imageUrl = product.querySelector('.s-image')?.src || 'No Image URL';

      products.push({
        title,
        rating,
        reviews,
        imageUrl,
      });
    });

    return products;

  } catch (error) {
    console.error('Error fetching data:', error);
    throw new Error('Error scraping Amazon');
  }
}

// API endpoint to trigger the scraping
app.get('/api/scrape', async (req, res) => {
  const { keyword } = req.query;
  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const products = await scrapeAmazon(keyword);
    res.json(products);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Start the server
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});


# Frontend Setup (HTML, CSS, and Vanilla JS with Vite)

# Step 1: Initialize the Frontend Project with Vite
npm create vite@latest amazon-scraper-frontend --template vanilla

# Step 2: Install Dependencies for the Frontend
cd amazon-scraper-frontend
npm install

# Step 3: Frontend Code (index.html)

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Amazon Scraper</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      max-width: 800px;
      margin: 0 auto;
    }
    input, button {
      padding: 10px;
      font-size: 16px;
    }
    .results {
      margin-top: 20px;
    }
    .product {
      border: 1px solid #ddd;
      margin-bottom: 10px;
      padding: 10px;
    }
    .product img {
      max-width: 100px;
    }
    .product h3 {
      margin: 0;
    }
    .product p {
      margin: 5px 0;
    }
  </style>
</head>
<body>

  <h1>Amazon Scraper</h1>
  <input type="text" id="keyword" placeholder="Enter a keyword" />
  <button id="searchButton">Search</button>

  <div class="results" id="results"></div>

  <script src="/src/main.js"></script>
</body>
</html>

# Step 4: Frontend JavaScript (src/main.js)

document.getElementById('searchButton').addEventListener('click', async () => {
  const keyword = document.getElementById('keyword').value;
  if (!keyword) {
    alert('Please enter a keyword.');
    return;
  }

  try {
    // Fetch data from the backend API
    const response = await fetch(`/api/scrape?keyword=${keyword}`);
    const data = await response.json();

    // Display the results
    const resultsDiv = document.getElementById('results');
    resultsDiv.innerHTML = '';

    if (data.error) {
      resultsDiv.innerHTML = `<p>Error: ${data.error}</p>`;
    } else {
      data.forEach(product => {
        const productDiv = document.createElement('div');
        productDiv.classList.add('product');

        productDiv.innerHTML = `
          <img src="${product.imageUrl}" alt="${product.title}">
          <h3>${product.title}</h3>
          <p>Rating: ${product.rating}</p>
          <p>Reviews: ${product.reviews}</p>
        `;

        resultsDiv.appendChild(productDiv);
      });
    }
  } catch (error) {
    console.error('Error fetching data:', error);
  }
});

# Step 5: Running the Application

# Backend: Start the Bun server (in amazon-scraper directory)
bun server src/server.js

# Frontend: Start the Vite development server (in amazon-scraper-frontend directory)
npm run dev

# Application should be available at http://localhost:3000
