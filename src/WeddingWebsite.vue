<script>
export default {
  name: 'WeddingWebsite',
  data() {
    return {
      currentPage: 'welcome',
      weddingData: {
        name1: 'Mack',
        name2: 'Jazmyn',
        date: 'J12025',
        location: 'Location1'
      }
    }
  },

  methods: {
    changePage(page) {
      this.currentPage = page;
      // Scroll to top when changing pages
      window.scrollTo(0, 0);
    },

    handleRsvpJump() {
      this.changePage('rsvp');
    }

  }
}
</script>

<template>
  <div class="wedding-app">

    <header class="nav-header">
      <nav class="nav-menu">
        <ul class="nav-links">
          <li :class="{ active: currentPage === 'welcome' }">
            <a href="#" @click.prevent="changePage('welcome')">Welcome</a>
          </li>
          <li :class="{ active: currentPage === 'rsvp' }">
            <a href="#" @click.prevent="changePage('rsvp')">RSVP</a>
          </li>
          <li :class="{ active: currentPage === 'the-day' }">
            <a href="#" @click.prevent="changePage('the-day')">The Day</a>
          </li>
          <li :class="{ active: currentPage === 'attire' }">
            <a href="#" @click.prevent="changePage('attire')">Attire</a>
          </li>
        </ul>
      </nav>
    </header>

    <main class="content">
      <!-- Welcome Page -->
      <div v-if="currentPage === 'welcome'" class="page landing-page">

        <!-- Banner with names and date -->
        <section class="banner">
          <div class="banner-overlay">
            <h1>{{ weddingData.name1 }} & {{ weddingData.name2 }}</h1>
            <p class="wedding-date">{{ weddingData.date }}</p>
            <p class="wedding-location">{{ weddingData.location }}</p>
          </div>
        </section>

        <!-- Content sections with alternating photos -->
        <section class="content-section">
          <div class="section-container">
            <div class="photo-container">
              <img src="/src/assets/finley-vertical1.jpg" alt="Couple photo" class="section-photo">
            </div>
            <div class="text-container">
              <h2>Our Story</h2>
              <p>text</p>
            </div>
          </div>
        </section>

        <section class="content-section">
          <div class="section-container reverse">
            <div class="text-container">
              <h2>Join Us</h2>
              <p> </p>
            </div>
            <div class="photo-container">
              <img src="/src/assets/finley-vertical-lake.jpg" alt="Venue photo" class="section-photo">
            </div>
          </div>
        </section>

        <button class="jump-to-rsvp-button" @click="handleRsvpJump">I'm Ready to RSVP</button>

      </div>

      <!-- RSVP Page (placeholder) -->
      <div v-else-if="currentPage === 'rsvp'" class="page rsvp-page">
        <h1>RSVP</h1>
        <p>Please let us know if you can attend our special day.</p>
        <!-- RSVP form later here -->
      </div>

      <!-- The Day Page (placeholder) -->
      <div v-else-if="currentPage === 'the-day'" class="page the-day-page">
        <h1>The Day</h1>
        <p>Schedule and details for our wedding day.</p>
        <!-- Schedule/details -->
      </div>

      <!-- Attire Page (placeholder) -->
      <div v-else-if="currentPage === 'attire'" class="page attire-page">
        <h1>Attire</h1>
        <p>Information about the dress code for our wedding.</p>
        <!-- Attire details -->
      </div>
    </main>
  </div>
</template>

<style>
:root {
  --primary-color: #d8b4fe;
  --text-color: #333;
  --background-color: #fff;
  --spacing-sm: 1rem;
  --spacing-md: 2rem;
  --spacing-lg: 4rem;
  --spacing-xl: 10rem;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  /* TODO font change? */
  font-family: 'Helvetica Neue', Arial, sans-serif;
  color: var(--text-color);
  background-color: var(--background-color);
  line-height: 1.6;
}

#app {
  min-width: 100vw;
  padding: 0;
}

/* Navigation menu styles */
.nav-header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background-color: rgba(255, 255, 255, 0.05);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 999;
}

.nav-menu {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem 2rem;
  margin: 0 auto;
}

.nav-links {
  display: flex;
  list-style: none;
}

.nav-links li {
  margin-left: 0.75rem;
  position: relative;
}

.nav-links a {
  text-decoration: none;
  color: var(--text-color);
  font-weight: 500;
  transition: color 0.3s ease;
}

.nav-links a:hover,
.nav-links .active a {
  color: var(--primary-color);
}

.nav-links .active::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 0;
  width: 100%;
  height: 2px;
  background-color: var(--primary-color);
}

/* Main styles */

.page {
  min-height: calc(100vh - 60px);
}

/* Banner styles */
.banner {
  width: 100vw;
  height: 100vh;
  margin-left: calc(-50vw + 50%);
  background-image: url('/src/assets/banner-lift.jpg');
  background-size: cover;
  background-position: center;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: white;
}

.banner::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
}

.banner-overlay {
  position: relative;
  z-index: 1;
  padding: 2rem;
}

.banner h1 {
  font-size: 3.5rem;
  margin-bottom: 1rem;
  font-weight: 300;
  letter-spacing: 2px;
}

.wedding-date {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
}

.wedding-location {
  font-size: 1.25rem;
}

/* Content sections */
.content-section {
  padding: var(--spacing-lg) var(--spacing-xl);

}

.section-container {
  display: flex;
  align-items: center;
  gap: var(--spacing-lg);
}

.photo-container,
.text-container {
  flex: 1;
}

.section-photo {
  width: 100%;
  height: auto;
  max-height: 110vh;
  object-fit: contain;
}

.text-container h2 {
  font-size: 2rem;
  margin-bottom: var(--spacing-sm);
  color: var(--primary-color);
}

.text-container p {
  font-size: 1.1rem;
  line-height: 1.8;
}

.jump-to-rsvp-button {
  background-color: #556b2f;
  font-size: 38px;
  width: 100%;
  border-radius: 0;
}

/* Page-specific styles */
.page:not(.landing-page) {
  padding: var(--spacing-lg);
  margin: 0 auto;
}
</style>
