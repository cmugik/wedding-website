<script setup>
import { computed, ref } from 'vue';

// State

const currentPage = ref('welcome');
const weddingData = ref({
  name1: 'Mack',
  name2: 'Jazmyn',
  date: 'August 17, 2025',
  location: 'Princeton, BC'
});

const formData = ref({
  name: '',
  email: '',
  phone: '',
  notes: '',
});

const errors = ref({
  name: false,
  contact: false
});

const formSubmitted = ref(false);

// Methods

const changePage = (page) => {
  currentPage.value = page;
  // Scroll to top when changing pages
  window.scrollTo(0, 0);
};

const handleRsvpJump = () => {
  changePage('rsvp');
};

const isValidEmail = computed(() => {
  if (!formData.value.email) return false;
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(formData.value.email);
});

const isValidPhone = computed(() => {
  if (!formData.value.phone) return false;
  const phoneRegex = /^\d{10,}$/;
  return phoneRegex.test(formData.value.phone.replace(/\D/g, ''));
});

const isFormValid = computed(() => {
  return formData.value.name && (isValidEmail.value || isValidPhone.value)
});

function validateForm() {
  errors.value.name = !formData.value.name;
  errors.value.contact = !(
    (formData.value.email && isValidEmail.value) ||
    (formData.value.phone && isValidPhone.value)
  );
}

function submitForm() {
  // stub
  formSubmitted.value = true;
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

      <!-- Color bar on non welcome pages -->
      <div class="nav-bar-shadow"></div>
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

              <p>Mack and Jaz met on August 17th, 2017. Exactly eight years to the day of our wedding.</p>

              <p>When Mack found out Jazmyn was a drywaller too, it was love at first sight. What we didn’t know then
                was how intertwined our lives had been all along. Our dads had been friends for years, and our families
                knew each other long before we were born. There are even photos of us as babies playing together on
                family camping trips. To make it even more fated, our childhood best friends were siblings.</p>

              <p>The early days of our relationship were filled with shenanigans, laughter, and excitement. When we
                first met, we were inseparable, spending seven straight days together the first week we met. (Bonus
                points if you remember the name we gave to that week!)</p>

              <p>As we got to know each other better, enjoying our first summer together, we started kayaking as a way
                to explore new things. After seeing a waterfall from our kayak on the other end of the lake, we wondered
                where it must be coming from. Determined to find out, we went home that night and researched the area.
                First thing the next day we set off, with everything but the kitchen sink on our backs. We took a
                massive bag of life jackets to use as beds, oversized bed pillows from home, a small but heavy tent and
                an assortment of random items we didn’t end up using. Laughing the entire time about how ridiculous we
                looked with life jackets on our backs.</p>

              <p>We paddled across a lake and hiked up the waterfall. Upon reaching the top we felt like we walked into
                the meadows of a fairytale. A beautiful lake and 10 little waterfalls running through flowers all around
                it. We didn’t want the adventure to end, so we took a quick dip in the glacial lake, dropped our bags
                and continued across the meadows to the other side.</p>
            </div>
          </div>
        </section>

        <section class="content-section">
          <div class="section-container reverse">
            <div class="text-container">
              <h2>Join Us</h2>

              <p>The mosquitos were thick and the exhaustion was high, but we didn’t care. We both kept saying “just one
                more waterfall”. On the journey through what felt like untouched nature, with no signs of humans, we
                found a tiny piece of hard plastic, which Jaz still has as a souvenir of the day.</p>

              <p>From that point on, we were hooked—whether it’s summiting mountains, scuba diving, or ski touring,
                adventure has been at the heart of our story.</p>

              <p>For our honeymoon, we’ll be road-tripping with Finnley through some of the most beautiful parks in the
                Yukon and Alaska.</p>

              <p>Thank you for celebrating with us on top of a mountain, in a way that feels so perfectly us.</p>
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
        <h1>Please let us know if you can attend our special day.</h1>
        <div class="rsvp-form-image-section">
          <div class="rsvp-form">
            <form v-if="!formSubmitted" id="rsvpForm" @submit.prevent="submitForm">
              <div class="form-group">
                <input type="text" id="name" v-model="formData.name" placeholder="Your name"
                  :class="{ 'highlight-error': errors.name }" @input="validateForm">
                <div v-if="errors.name" class="error">Please include your name!</div>
              </div>
              <div class="form-group">
                <div class="contact-info">
                  <input type="email" id="email" v-model="formData.email" placeholder="Email"
                    :class="{ 'highlight-error': errors.contact && formData.email && !isValidEmail }"
                    @input="validateForm">
                  <input type="tel" id="phone" v-model="formData.phone" placeholder="Phone #"
                    :class="{ 'highlight-error': errors.contact && formData.phone && !isValidPhone }"
                    @input="validateForm">
                </div>
                <div v-if="errors.contact" class="error">Please provide at least one valid contact method!</div>
              </div>
              <div class="form-group">
                <textarea id="notes" v-model="formData.notes"
                  placeholder="Any special requests, accomodations or information we should know?"></textarea>
              </div>
              <div class="rsvp-button">
                <button type="submit" id="submitBtn" :disabled="!isFormValid">Send RSVP</button>
              </div>
            </form>
            <div v-if="formSubmitted" class="resolution-message">
              Thank you for your RSVP! We've received your information.
            </div>
          </div>
          <div class="rsvp-image">
            <img src="./assets/vertical-champagne.jpg">
          </div>
        </div>
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
/* Potential Palettes

#7D9D7F (Sage Green)
#F5F1E6 (Ivory/Off-White)
#E0CDAD (Beige)
#717171 (Medium Gray)
#373737 (Charcoal Gray)

#7D9D7F (Sage Green)
#800020 (Burgundy)
#F0EDE5 (Light Neutral)
#4A4A4A (Dark Gray)

#7D9D7F (Sage Green)
#18304A (Navy Blue)
#EEEEE4 (Off-White)
#BFA78B (Light Taupe)

#7D9D7F (Sage Green)
#D4A82C (Mustard Yellow)
#F9F5EB (Cream)
#54483E (Dark Brown)

#7D9D7F (Sage Green)
#C39E9E (Dusty Rose)
#F3F1EE (Off-White)
#B6B6B4 (Light Gray)

#7D9D7F (Sage Green)
#B28FB0 (Mauve)
#EDE8F0 (Pale Lavender White)
#CEBEB0 (Light Taupe)

#7D9D7F (Sage Green)
#B3C7D6 (Pale Blue)
#F5F7F8 (Blue-White)
#D1CAC1 (Light Greige)

#7D9D7F (Sage Green)
#C56434 (Terracotta)
#EBE3D7 (Cream)
#6D5D4B (Brown)

#7D9D7F (Medium Sage Green)
#2A4D3E (Dark Forest Green)
#F0EDE4 (Off-White)
#A39079 (Taupe)

#7D9D7F (Sage Green)
#9F7A60 (Caramel)
#5E4B3E (Chestnut)
#F2EBE3 (Cream)
*/

:root {
  --primary-color: #737373;
  --secondary-color: #a9bfb3;
  --background-color: #F5F1E6;
  --text-color: #373737;
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
  padding: 1rem 1rem;
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
  color: var(--primary-color);
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

.nav-bar-shadow {
  width: 100%;
  height: 58px;
  top: 0;
  left: 0;
  position: absolute;
  background-color: var(--secondary-color);

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

.section-photo {
  width: 100%;
  height: auto;
  max-height: 110vh;
  object-fit: contain;
}

.photo-container,
.text-container {
  flex: 1;
}

.text-container h2 {
  font-size: 2rem;
  margin-bottom: var(--spacing-sm);
  color: var(--primary-color);
}

.text-container p {
  font-size: 1.3rem;
  line-height: 1.8;
  margin-bottom: 12px;
}

.jump-to-rsvp-button {
  background-color: var(--secondary-color);
  font-size: 38px;
  width: 100%;
  border-radius: 0;
  color: var(--text-color);
}

/* Page-specific styles */
.page:not(.landing-page) {
  padding: var(--spacing-lg);
  margin: 0 auto;
}

.rsvp-page {
  position: relative;
  padding-top: 2rem;
}

/* RSVP Page specific style */

.rsvp-form-image-section {
  display: flex;
  gap: var(--spacing-md);
  margin-top: var(--spacing-md);
  margin-bottom: 0px;
}

.form-group input[type="text"],
.form-group textarea {
  width: 100%;
  box-sizing: border-box;
  padding: 10px;
  margin-bottom: 1px;
}

.form-group textarea {
  resize: vertical;
}

.form-group input {
  height: 80px;
  font-size: 30px;
}

.rsvp-button button {
  width: 100%;
}

.contact-info {
  display: flex;
}

.contact-info input {
  flex: 1;
  /* Make both inputs take equal space */
  box-sizing: border-box;
  padding: 10px;
}

.rsvp-form {
  flex: 1;
  border-radius: 8px;
}

.rsvp-image {
  width: 50%;
  /* Set to exactly 50% of the parent container */
  max-width: 50vw;
  /* Max width of 50% of viewport */
  display: flex;
  align-items: center;
  justify-content: center;
}

.rsvp-image img {
  width: 100%;
  height: auto;
  max-height: 90vh;
  /* Prevent image from being too tall */
  object-fit: contain;
  /* Changed from cover to maintain aspect ratio */
  border-radius: 8px;
}

/* Responsive design */
@media (max-width: 768px) {
  .rsvp-form-image-section {
    flex-direction: column;
  }

  .rsvp-image {
    order: -1;
    /* Move image above form on mobile */
    width: 100%;
    max-width: 100%;
    margin-bottom: var(--spacing-md);
  }
}
</style>
