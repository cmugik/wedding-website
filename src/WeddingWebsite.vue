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
  contact: false,
  attendance: false
});

const formSubmitted = ref(false);

const formAttempted = ref(false);

// Methods

const changePage = (page) => {
  currentPage.value = page;
  // Scroll to top when changing pages
  window.scrollTo(0, 0);
};

const handleDayJump = () => {
  changePage('the-day');
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
  return !errors.value.name && !errors.value.contact && !errors.value.attendance;
});

function validateForm() {
  errors.value
  errors.value.name = !formData.value.name;
  errors.value.contact = !(
    (formData.value.email && isValidEmail.value) ||
    (formData.value.phone && isValidPhone.value)
  );
  errors.value.attendance = !document.querySelector('input[name="rsvp"]:checked');
}

async function submitForm() {
  formAttempted.value = true;
  validateForm();

  if (!isFormValid.value) {
    return;
  }

  try {
    const rsvpValue = document.querySelector('input[name="rsvp"]:checked')?.value;

    if (!rsvpValue) {
      return;
    }

    const dataToSend = {
      rsvp: rsvpValue,
      name: formData.value.name,
      email: formData.value.email,
      phone: formData.value.phone,
      notes: formData.value.notes,
      timestamp: new Date().toISOString()
    };

    const submitBtn = document.getElementById('submitBtn');
    submitBtn.disabled = true;
    submitBtn.textContent = 'Sending...';

    // Send to Cloudflare Worker
    const response = await fetch('https://rough-sun-8ed1.caleb-kellett.workers.dev/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(dataToSend),
    });

    if (!response.ok) {
      console.error(response);
      throw new Error('Failed to submit RSVP');
    }

    formSubmitted.value = true;
  } catch (error) {
    console.error('Error submitting RSVP:', error);
    alert('There was an error submitting your RSVP. Please try again later.');
  } finally {
    if (!formSubmitted.value) {
      const submitBtn = document.getElementById('submitBtn');
      submitBtn.disabled = false;
      submitBtn.textContent = 'Send RSVP';
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
              <img src="/src/assets/vertical-handholds-finley.jpg" alt="Couple photo" class="section-photo">
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
              <img src="/src/assets/vertical-kalm-rock.jpg" alt="Venue photo" class="section-photo">
            </div>
          </div>
        </section>

        <button class="jump-to-day-button" @click="handleDayJump">Tell Me About The Day</button>

      </div>

      <!-- RSVP Page -->

      <div v-else-if="currentPage === 'rsvp'" class="page rsvp-page">
        <div class="rsvp-deadline-header">
          <h1>
            Please let us know if you can attend our special day before June 1st!
          </h1>
        </div>
        <div class="rsvp-form-image-section">
          <div class="rsvp-form">
            <form v-if="!formSubmitted" id="rsvpForm" @submit.prevent="submitForm">
              <div class="rsvp-options">
                <label class="rsvp-option rsvp-option-yes">
                  <input type="radio" name="rsvp" value="yes" @change="errors.attendance = false">
                  <span class="custom-radio"></span>
                  <span class="rsvp-option-text">Yes, I can attend</span>
                </label>
                <label class="rsvp-option rsvp-option-no">
                  <input type="radio" name="rsvp" value="no" @change="errors.attendance = false">
                  <span class="custom-radio"></span>
                  <span class="rsvp-option-text">No, I cannot attend</span>
                </label>
                <div v-if="formAttempted && errors.attendance" class="error">Please select whether you can attend or
                  not!</div>
              </div>

              <div class="form-group">
                <input type="text" id="name" v-model="formData.name" @input="errors.name = false"
                  placeholder="Your name" :class="{ 'highlight-error': formAttempted && errors.name }">
                <div v-if="formAttempted && errors.name" class="error">Please include your name!</div>
              </div>

              <div class="form-group">
                <div class="contact-info">
                  <input type="email" id="email" v-model="formData.email" @input="errors.contact = false"
                    placeholder="Email"
                    :class="{ 'highlight-error': formAttempted && errors.contact && formData.email && !isValidEmail }">
                  <input type="tel" id="phone" v-model="formData.phone" @input="errors.contact = false"
                    placeholder="Phone #"
                    :class="{ 'highlight-error': formAttempted && errors.contact && formData.phone && !isValidPhone }">
                </div>
                <div v-if="formAttempted && errors.contact" class="error">Please provide at least one valid contact
                  method!</div>
              </div>
              <div class="form-group">
                <textarea id="notes" v-model="formData.notes"
                  placeholder="Any accomodation requests or information we should know?"></textarea>
              </div>
              <div class="form-group-rsvp-button">
                <button type="submit" id="submitBtn" :disabled="!isFormValid">Send RSVP</button>
              </div>
            </form>
            <div v-if="formSubmitted" class="resolution-message">
              Thank you for your RSVP! We've received your information.
              <br>
              If you would like to get us a gift, please consider
              adding to our home or honeymoon fund! We're planning on buying a house in the near future. You can send
              these to kellettjazmyn@gmail.com, or bring cash to the
              wedding. If you prefer a tangible gift, our registry will be live in April!
            </div>
            <div class="rsvp-spacing-bar">
            </div>
          </div>
          <div class="rsvp-image">
            <img src="./assets/vertical-champagne-cropped.jpg">
          </div>
        </div>
      </div>


      <!-- The Day Page -->
      <div v-else-if="currentPage === 'the-day'" class="page the-day-page">


        <div class="the-day-content">
          <div class="the-day-text">
            <div class="location-text">
              <h2>Location</h2>

              <p>
                Our wedding will be held at
                <a href="https://jurasunflowersummit.ca/">Jura Sunflower Summit</a>,
                a stunning 200-acre mountaintop venue surrounded by untouched
                crown land forest as far as the eye can see.
                The location is accessible via a well-maintained forest service road. However, low-clearance vehicles
                (such as sports cars) may not be able to make it to the top, so we recommend using vehicles with higher
                clearance.
              </p>

              <p>
                The entrance to the forest service road is located approximately 12 minutes outside of Princeton, BC, at
                3362 Hembrie Mountain Road — using a GPS/maps app is recommended.
                Once on the forest service road, you will see signs guiding you all the way up to the venue. If you
                require
                assistance with transportation or accessing the site, please let us know when you RSVP.
                We will be sharing more detailed access information at the end of May.</p>
              <p>
                In the meantime, if you have any questions, feel free to message the bride at
                <a href="tel:604-902-0397">604-902-0397</a>.
              </p>

            </div>

            <div class="accommodations-text">

              <h2>Accommodations</h2>

              <p>
                Our venue offers on-site camping for anyone who would like to stay close to the celebration! For our BC
                guests, please bring your usual camping gear. Trailers, campers, rooftop tents, and traditional tents
                are
                all welcome.
              </p>

              <p>To reserve a site ahead of time, please follow the links provided below.
                If you’re flying in and would like to camp but won’t be able to bring your gear, just let us know when
                you
                RSVP
                or contact the bride directly at
                <a href="tel:604-902-0397">604-902-0397</a>.
              </p>

              <p>We also have a few alternative accommodations available, but they are limited, so please reach out as
                early
                as
                possible if you would like to secure one.</p>

              <p>
                If camping isn’t your style but you’d still like to stay nearby, there is a Sandman Inn located in
                Princeton, as well as a few motels.
              </p>

              <h3>Suggested Items to Bring:</h3>

              <ul>
                <li>Tent, camper, or rooftop tent</li>
                <li>Sleeping pad</li>
                <li>Sleeping bags or blankets</li>
                <li>Pillows</li>
                <li>Snacks and breakfast for the next morning</li>
                <li>Warm layers for the evening (the site sits at 1,200m / 4,000ft and can get chilly at night)</li>
                <li>Propane fires will be provided, but feel free to bring your own if you’d like</li>
              </ul>


            </div>


            <div class="timeline-text">
              <h2>Timeline</h2>

              <p>
                <strong>Wedding Party:</strong>
                If you are part of the wedding party, you will receive your personalized day-of timeline directly from
                the
                Bride & Groom.
              </p>

              <p>
                <strong>All Guests:</strong>
                Please plan to arrive between 3:00 PM and 3:45 PM. Below is a general outline of the day’s events:
              </p>

              <ul>
                <li><strong>3:00 PM – 3:45 PM</strong> - Welcome drinks & lawn games</li>
                <li><strong>4:00 PM – 5:00 PM</strong> - Ceremony</li>
                <li><strong>5:00 PM – 5:45 PM</strong> - Champagne toast & a short break</li>
                <li><strong>5:45 PM – 7:00 PM</strong> - Dinner</li>
                <li><strong>7:10 PM</strong> - First dance</li>
                <li><strong>7:20 PM</strong> - Father-daughter dance</li>
                <li><strong>7:30 PM</strong> - Cake cutting</li>
                <li><strong>7:45 PM – late</strong> - Bonfire, s’mores & dancing under the stars</li>
              </ul>

            </div>


            <div class="food-drink-text">
              <h2>Food & Drinks</h2>

              <p>
                We will be serving a BBQ-style buffet with two meats and a variety of hearty side dishes. Portions will
                be
                generous, so come hungry!
                Following dinner, we’ll have wedding cake and a s’mores station to satisfy your sweet tooth.
                and
                a champagne toast for all guests following the ceremony.
              </p>

              <p>
                Beyond that, please note that alcohol will be limited to a couple of drinks per person. If you’d like to
                enjoy more, feel free to bring your favorite beverages to enjoy throughout the evening.
              </p>
            </div>

          </div>

          <div class="the-day-image">
            <img src="./assets/vertical-cake.jpg">
          </div>
        </div>

      </div>

      <!-- Attire Page -->
      <div v-else-if="currentPage === 'attire'" class="page attire-page">
        <div class="attire-content">
          <div class="attire-text">
            <p> We’re embracing the magic of nature for our wedding—think autumn inspired.
              Picture rich fall colors: burnt orange, deep purple, maroon, browns, reds, and gold, all paired with
              semi-formal
              attire. </p>
            <p> For the ladies: mid-to-floor length dresses, a sharp pantsuit, or a dressy jumpsuit will fit right in.
              For the gents: darker tones are your friend. A suit, a blazer with dress pants, or even just a nice shirt
              with
              slacks will do the trick. </p>
            <p> Bonus tip! Once the sun goes down, we’ll be swapping dancing shoes for cozy layers. At 1200m (4000ft),
              the
              evening air is a bit chilly —even in summer. So pack your favorite sweaters, blanket, jacket, warm socks,
              and
              toque. We’ll have a fire (or a propane one if Mother Nature’s feeling spicy), and plenty of time under the
              stars. </p>
          </div>
          <div class="attire-sidebar"> </div>
        </div>
        <div class="attire-footbar"> </div>
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
  color: var(--second-color);
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

.jump-to-day-button {
  background-color: var(--secondary-color);
  font-size: 38px;
  width: 100%;
  border-radius: 0;
  color: var(--text-color);
}

/* Page-specific styles */
.page:not(.landing-page, .the-day-page, .attire-page) {
  padding: var(--spacing-lg);
}

.rsvp-page {
  position: relative;
  padding-top: 2rem;
}

.the-day-page {
  padding-top: var(--spacing-lg);
  padding-bottom: var(--spacing-lg);
  padding-left: var(--spacing-lg);
  padding-right: var(--spacing-sm);
}

.attire-page {
  min-height: calc(100vh - 60px);
  padding-top: 56px;
}


/* RSVP Page specific style */

.rsvp-deadline-header {
  margin-top: 1rem;
  margin-bottom: 2rem;
  color: var(--primary-color);
}

.rsvp-form-image-section {
  display: flex;
  gap: var(--spacing-md);
  margin-bottom: 0px;
  align-items: center;
}

.rsvp-form {
  flex: 1;
  border-radius: 8px;
}

.form-group input[type="text"],
.form-group textarea {
  width: 100%;
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

/* Setting rsvp form border and background for inputs & button */
.form-group input[type="text"],
.form-group textarea,
.contact-info input,
.form-group-rsvp-button button {
  background: rgba(0, 0, 0, 0.025);
  border-width: 1px;
  border-color: var(--primary-color);
  margin-top: 1px;
  margin-bottom: 1px;
  outline: none;
  color: var(--primary-color);
}

.form-group input[type="text"]:focus,
.form-group textarea:focus,
.contact-info input:focus,
.form-group-rsvp-button button:focus {
  box-shadow: 0 0 7px rgba(75, 150, 225, 0.5);
}

.form-group-rsvp-button button {
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

.rsvp-options {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 1rem;
}

.rsvp-options.highlight-error {
  border: 1px solid red;
  background-color: rgba(255, 0, 0, 0.05);
  padding: 10px;
  border-radius: 5px;
}

.rsvp-option {
  display: flex;
  align-items: center;
  padding: 8px;
  border-radius: 3px;
  cursor: pointer;
  border: 1px solid var(--primary-color);
  transition: all 0.3s ease;
  background: rgba(0, 0, 0, 0.025);
}

.rsvp-option:hover {
  background-color: rgba(100, 100, 100, 0.1);
}

.rsvp-option input[type="radio"] {
  display: none;
}

.custom-radio {
  display: inline-flex;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  border: 2px solid var(--primary-color);
  border-radius: 50%;
  margin-right: 10px;
}

.rsvp-option-yes input[type="radio"]:checked+.custom-radio:after {
  content: '✓';
}

.rsvp-option-no input[type="radio"]:checked+.custom-radio:after {
  content: '✗';
}

.rsvp-image {
  width: 50%;
  max-width: 50vw;
  display: flex;
  align-items: center;
  justify-content: center;
}

.rsvp-image img {
  width: 100%;
  height: auto;
  max-height: 90vh;
  object-fit: contain;
  border-radius: 8px;
}

.rsvp-spacing-bar {
  height: 90px;
}

/* the-day specific design */

.the-day-content {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  gap: var(--spacing-md);
  align-items: center;
}

.the-day-text {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.the-day-image img {
  max-width: 30vw;
  height: auto;
}

.location-text {
  justify-content: left;
  align-items: start;
  text-align: start;
}

.accommodations-text {
  justify-content: left;
  align-items: start;
  text-align: start;
}

.timeline-text {
  justify-content: left;
  align-items: start;
  text-align: start;
}

.food-drink-text {
  justify-content: left;
  align-items: start;
  text-align: start;
}

/* attire-page specific styles */

.attire-content {
  display: flex;
  min-height: calc(100vh - 56px - var(--spacing-lg));
}

.attire-text {
  flex: 1;
  padding-right: var(--spacing-md);
  font-size: 1.3rem;
  line-height: 1.8;
  padding-top: var(--spacing-md);
  padding-left: var(--spacing-lg);
}

.attire-text p {
  text-align: left;
}

.attire-sidebar {
  width: var(--spacing-lg);
  background-color: var(--secondary-color);
  z-index: 1003;
}

.attire-footbar {
  height: var(--spacing-lg);
  width: 100%;
  background-color: var(--secondary-color);
  margin-top: auto;
}

/* Responsive design - expanding on existing code */
@media (max-width: 768px) {

  /* Your existing responsive code */
  .rsvp-form-image-section {
    flex-direction: column;
  }

  .rsvp-image {
    order: -1;
    width: 100%;
    max-width: 100%;
    margin-bottom: var(--spacing-md);
  }

  /* Additional responsive improvements */
  /* Navigation adjustments */
  .nav-menu {
    padding: 0.5rem;
  }

  .nav-links {
    justify-content: center;
    width: 100%;
  }

  .nav-links li {
    margin-left: 0.4rem;
    font-size: 0.9rem;
  }

  /* Banner text size */
  .banner h1 {
    font-size: 2.2rem;
  }

  .wedding-date {
    font-size: 1.2rem;
  }

  .wedding-location {
    font-size: 1rem;
  }

  /* Content sections stack */
  .section-container,
  .section-container.reverse {
    flex-direction: column;
  }

  .content-section {
    padding: var(--spacing-md) var(--spacing-sm);
  }

  .text-container p {
    font-size: 1.1rem;
    line-height: 1.6;
  }

  /* The day page mobile layout */
  .the-day-content {
    flex-direction: column;
  }

  .the-day-image img {
    max-width: 100%;
    margin-top: 20px;
  }

  /* Attire page adjustments */
  .attire-content {
    flex-direction: column;
  }

  .attire-sidebar {
    width: 100%;
    height: var(--spacing-sm);
  }

  .attire-text {
    padding: var(--spacing-sm);
  }

  /* Form improvements */
  .contact-info {
    flex-direction: column;
  }

  .contact-info input {
    margin-bottom: 5px;
  }

  .form-group input {
    height: 50px;
    font-size: 18px;
  }

  .jump-to-day-button {
    font-size: 28px;
    padding: 10px;
  }
}
</style>
