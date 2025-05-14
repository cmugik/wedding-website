<script setup>
import { computed, ref, onMounted } from 'vue';

// State

const currentPage = ref('welcome');
const weddingData = ref({
  name1: 'MACK',
  name2: 'JAZMYN',
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

// Registry state
const items = ref([]);
const selectedItems = ref([]);
const showModal = ref(false);
const userName = ref('');
const userEmail = ref('');
const successMessage = ref('');
const errorMessage = ref('');
const isSubmitting = ref(false);
const isLoading = ref(false);
const reservationButtonText = computed(() =>
  selectedItems.value.length <= 1 ? "I'll get this" : "I'll get these"
);

// Fetch registry items from backend
async function fetchRegistry() {
  isLoading.value = true;
  try {
    const res = await fetch('https://rough-sun-8ed1.caleb-kellett.workers.dev/registry');
    if (!res.ok) throw new Error('Failed to fetch registry');
    const data = await res.json();
    console.log(data);
    items.value = data.items || [];
  } catch (e) {
    errorMessage.value = 'Could not load registry items.';
  } finally {
    isLoading.value = false;
    console.log(items);
  }
}

// Call fetchRegistry when component mounts
onMounted(fetchRegistry);

// Only show items with at least one open purchaser slot
const availableItems = computed(() =>
  items.value.filter(
    (item) =>
      (!item.Purchaser1Mail || !item.Purchaser1Name) ||
      (!item.Purchaser2Mail || !item.Purchaser2Name)
  )
);

async function submitReservation() {
  isSubmitting.value = true;
  errorMessage.value = '';
  successMessage.value = '';

  // Prepare payload
  const selectedItemObjs = items.value
    .filter((item) => selectedItems.value.includes(item.ItemName))
    .map((item) => ({
      ItemName: item.ItemName,
    }));

  const payload = {
    items: selectedItemObjs,
    name: userName.value,
    email: userEmail.value,
  };

  try {
    const res = await fetch('https://rough-sun-8ed1.caleb-kellett.workers.dev/registry', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });

    if (!res.ok) {
      throw new Error('Failed to reserve items, try again later.');
    }

    successMessage.value = `Thank you, ${userName.value}! Your selection has been saved.`;
    showModal.value = false;
    userName.value = '';
    userEmail.value = '';
    selectedItems.value = [];
    await fetchRegistry(); // Refresh the registry list
  } catch (e) {
    errorMessage.value = 'There was a problem saving your selection. Please try again.';
    successMessage.value = '';
  } finally {
    isSubmitting.value = false;
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
          <li :class="{ active: currentPage === 'gift' }">
            <a href="#" @click.prevent="changePage('gift')">Gifts</a>
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
            <h3 class="wedding-date">{{ weddingData.date }}</h3>
            <h3 class="wedding-location">{{ weddingData.location }}</h3>
          </div>
        </section>

        <!-- Content sections with alternating photos -->
        <section class="content-section">
          <div class="section-container">
            <div class="photo-container">
              <figure>
                <img src="/src/assets/vertical-kalm-rock.jpg" alt="Couple photo" class="section-photo hidden-on-mobile">
                <figcaption> photos by alejandroconnor.com </figcaption>
              </figure>
            </div>
            <div class="text-container">
              <h2>Our Story</h2>

              <p>Mack and Jaz met on August 17th, 2017. Exactly eight years to the day of our wedding.</p>

              <p>When Mack found out Jazmyn was a drywaller too, it was love at first sight. What we didn't know then
                was how intertwined our lives had been all along. Our dads had been friends for years, and our families
                knew each other long before we were born. There are even photos of us as babies playing together on
                family camping trips. To make it even more fated, our childhood best friends were siblings.</p>

              <p>The early days of our relationship were filled with shenanigans, laughter, and excitement. When we
                first met, we were inseparable, spending seven straight days together the first week we met. (Bonus
                points if you remember the name we gave to that week!)</p>

              <p>As we got to know each other better, enjoying our first summer together, we started kayaking as a way
                to explore new things. After seeing a waterfall from our kayak on the other end of the lake, we wondered
                where it must be coming from.</p>

              <p>Determined to find out, we went home that night and researched the area. First thing the next day we
                set off, with everything but the kitchen sink on our backs. We took a massive bag of life jackets to use
                as beds, oversized bed pillows from home, a small but heavy tent and an assortment of random items we
                didn't end up using. Laughing the entire time about how ridiculous we looked with life jackets on our
                backs.</p>

              <p>We paddled across a lake and hiked up the waterfall. Upon reaching the top we felt like we walked into
                the meadows of a fairytale. A beautiful lake and 10 little waterfalls running through flowers all around
                it. We didn't want the adventure to end, so we took a quick dip in the glacial lake, dropped our bags
                and continued across the meadows to the other side.</p>
            </div>
          </div>
        </section>

        <section class="content-section">
          <div class="section-container reverse">
            <div class="text-container">

              <p>The mosquitos were thick and the exhaustion was high, but we didn't care. We both kept saying "just one
                more waterfall". On the journey through what felt like untouched nature, with no signs of humans, we
                found a tiny piece of hard plastic, which Jaz still has as a souvenir of the day.</p>

              <p>From that point on, we were hooked—whether it's summiting mountains, scuba diving, or ski touring,
                adventure has been at the heart of our story.</p>

              <p>That spot has been special to us for years—we've celebrated several anniversaries there, including our
                most recent one. It's also where Finnley went on his first big hike. Even our engagement photos, the
                ones you see on the website, were taken in that same area. It's one of those places that holds a lot of
                our favorite memories.</p>

              <p>For our honeymoon, we'll be road-tripping with Finnley through some of the most beautiful parks in the
                Yukon and Alaska.</p>

              <p>Thank you for celebrating with us on top of a mountain, in a way that feels so perfectly us.</p>
              <br>

              <h2>Join Us</h2>
            </div>
            <div class="photo-container">
              <figure>
                <img src="/src/assets/vertical-lift2.jpg" alt="Venue photo" class="section-photo">
                <figcaption> photos by alejandroconnor.com </figcaption>
              </figure>
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
                  placeholder="Any accommodation requests or information we should know?"></textarea>
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
            <figure>
              <img src="./assets/vertical-champagne-cropped.jpg">
              <figcaption> photos by alejandroconnor.com </figcaption>
            </figure>
          </div>
        </div>
      </div>


      <!-- The Day Page -->
      <div v-else-if="currentPage === 'the-day'" class="page the-day-page">
        <div class="the-day-content">
          <div class="the-day-text">

            <br>
            <div class="location-text">
              <h2 class="location-header">Location</h2>

              <br>

              <p>
                Our wedding will be held at
                <a href="https://jurasunflowersummit.ca/">Jura Sunflower Summit</a>,
                a stunning 200-acre mountaintop venue surrounded by untouched
                crown land forest as far as the eye can see.
              </p>

              <p>
                The location is accessible via a well-maintained forest service road. However, low-clearance vehicles
                (such as sports cars) may not be able to make it to the top, so we recommend using vehicles with higher
                clearance.
              </p>

              <p>
                The entrance to the forest service road is located approximately 12 minutes outside of Princeton, BC, at
                3362 Hembrie Mountain Road — using a GPS/maps app is recommended.
              </p>

              <p>
                Once on the forest service road, you will see signs guiding you all the way up to the venue.
              </p>

              <p>
                If you require assistance with transportation or accessing the site, please let us know when you RSVP.
              </p>

              <p>
                We will be sharing more detailed access information at the end of May.
              </p>

              <p>
                In the meantime, if you have any questions, feel free to message the bride at
                <a href="tel:604-902-0397">604-902-0397</a>.
              </p>

            </div>

            <div class="accommodations-text">

              <br>
              <h2>Accommodations</h2>
              <br>

              <p>
                Our venue offers on-site camping for anyone who would like to stay close to the celebration!
              </p>

              <p>
                For our BC guests, please bring your usual camping gear. Trailers, campers, rooftop tents, and
                traditional tents
                are all welcome.
              </p>

              <p>
                To reserve a site ahead of time, please follow the links provided below.
              </p>

              <p>
                If you're flying in and would like to camp but won't be able to bring your gear, just let us know when
                you RSVP or contact the bride directly at
                <a href="tel:604-902-0397">604-902-0397</a>.
              </p>

              <p>
                We also have a few alternative accommodations available, but they are limited, so please reach out as
                early as possible if you would like to secure one.
              </p>

              <br>
              <h3 class="day-list-header"> Suggested Items to Bring:</h3>

              <ul class="day-list">
                <li>Tent, camper, or rooftop tent</li>
                <li>Sleeping pad</li>
                <li>Sleeping bags or blankets</li>
                <li>Pillows</li>
                <li>Snacks and breakfast for the next morning</li>
                <li>Warm layers for the evening (the site sits at 1,200m / 4,000ft and can get chilly at night)</li>
                <li>Propane fires will be provided, but feel free to bring your own if you'd like</li>
              </ul>
              <br>

              <p>
                If camping isn't your style but you'd still like to stay nearby, there is a Sandman Inn located in
                Princeton municipality near by.
              </p>

            </div>

            <br>
            <div class="timeline-text">
              <br>
              <h2>Timeline</h2>
              <br>

              <p>
                <strong>Wedding Party:</strong> If you are part of the wedding party, you will receive your personalized
                day-of timeline directly from
                the Bride & Groom.
              </p>

              <p>
                <strong>All Guests:</strong> Please plan to arrive between 3:00 PM and 3:45 PM. Below is a general
                outline of the day's events:
              </p>

              <ul class="day-list">
                <li>
                  <div class="timeline-time"><strong>3:00 PM – 3:45 PM</strong></div>
                  <div class="timeline-event">Welcome drinks & lawn games</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>4:00 PM – 5:00 PM</strong></div>
                  <div class="timeline-event">Ceremony</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>5:00 PM – 5:45 PM</strong></div>
                  <div class="timeline-event">Champagne toast & a short break</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>5:45 PM – 7:00 PM</strong></div>
                  <div class="timeline-event">Dinner</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>7:10 PM</strong></div>
                  <div class="timeline-event">First dance</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>7:20 PM</strong></div>
                  <div class="timeline-event">Father-daughter dance</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>7:30 PM</strong></div>
                  <div class="timeline-event">Cake cutting</div>
                </li>
                <li>
                  <div class="timeline-time"><strong>7:45 PM – late</strong></div>
                  <div class="timeline-event">Bonfire, s'mores & dancing under the stars</div>
                </li>
              </ul>

              <br>

              <p>
                We can't wait to celebrate with you!!!
              </p>
            </div>

            <br>
            <div class="food-drink-text">
              <br>
              <h2>Food and Drinks</h2>
              <br>

              <p>
                We will be serving a BBQ-style buffet with two meats and a variety of hearty side dishes. Portions will
                be generous, so come hungry!
              </p>

              <p>
                Following dinner, we'll have wedding cake and a s'mores station to satisfy your sweet tooth.
              </p>

              <p>
                We'll also be doing a welcome drink upon arrival for our guests who arrive prior to ceremony start time,
                and a champagne toast for all guests following the ceremony. Beyond that, please note that alcohol will
                be limited to a couple of drinks per person. If you'd like to enjoy more, feel free to bring your
                favorite beverages to enjoy throughout the evening.
              </p>
            </div>

          </div>

        </div>

        <figure>
          <div class="the-day-image">
            <img src="./assets/vertical-cake.jpg">
          </div>
          <figcaption> photos by alejandroconnor.com </figcaption>
        </figure>

      </div>
      <!-- Attire Page -->
      <div v-else-if="currentPage === 'attire'" class="page attire-page">
        <div class="attire-content">
          <div class="attire-text">
            <h2> We’re embracing the magic of nature for our wedding—think autumn inspired.</h2>
            <br>
            <p>
              Picture rich fall colors: burnt orange, deep purple, maroon, browns, reds, and gold, all paired with
              semi-formal
              attire. </p>
            <p> For the ladies: mid-to-floor length dresses, a sharp pantsuit, or a dressy jumpsuit will fit right in.
            </p>
            <p>
              For the gents: darker tones are your friend. A suit, a blazer with dress pants, or even just a nice shirt
              with
              slacks will do the trick. </p>
            <p> Bonus tip! Once the sun goes down, we’ll be swapping dancing shoes for cozy layers. At 1200m (4000ft),
              the
              evening air is a bit chilly —even in summer. So pack your favorite sweaters, blanket, jacket, warm socks,
              and
              toque. </p>
            <p>We’ll have a fire (or a propane one if Mother Nature’s feeling spicy), and plenty of time under the
              stars. </p>
          </div>

          <div class="attire-image">
            <figure>
              <img src="/src/assets/vertical-lift1.jpg" class="attire-image">
              <figcaption> photos by alejandroconnor.com </figcaption>
            </figure>
          </div>

        </div>
        <div class="attire-footbar"> </div>
      </div>
      <div v-else-if="currentPage === 'gift'" class="page gift-page">
        <div class="registry-blurb">
          <h2 class="registry-header">Wedding Registry</h2>
          <p>
          The best present you can give us is your presence on our special day.
          If you do choose to bring a gift, please see the registry below for ideas.
          </p>
          <p>
          If you'd prefer to donate to our honeymoon or new home fund, our email is jazandmack@gmail.com
          </p>
        </div>
        <table>
          <thead>
            <tr>
              <th>Item</th>
              <th>Select</th>
            </tr>
          </thead>
          <tbody>
              <tr v-if="availableItems.length === 0">
                <td colspan="2">No items available</td>
              </tr>
              <tr v-for="item in availableItems" :key="item.ItemName">
                <td>
                  <a :href="item.ItemLink" target="_blank">{{ item.ItemName }}</a>
                </td>
                <td>
                  <input
                    type="checkbox"
                    :value="item.ItemName"
                    v-model="selectedItems"
                  />
                </td>
              </tr>
            </tbody>
        </table>
        <button
          style="margin-top: 1em"
          :disabled="selectedItems.length === 0"
          @click="showModal = true"
        >
        {{ reservationButtonText }}
        </button>

        <!-- Modal -->
        <div
          v-if="showModal"
          class="modal-overlay"
          @click.self="showModal = false"
        >
          <div class="modal-content">
            <h3>Let us know who you are</h3>
            <label>
              Name:
              <input v-model="userName" type="text" placeholder="Your name" />
            </label>
            <br />
            <label>
              Email:
              <input v-model="userEmail" type="email" placeholder="Your email" />
            </label>
            <br />
            <button
              style="margin-top: 1em"
              :disabled="!userName || !userEmail || isSubmitting"
              @click="submitReservation"
            >
              Submit
            </button>
            <button style="margin-left: 1em" @click="showModal = false">
              Cancel
            </button>
          </div>
        </div>
        <div v-if="successMessage" class="success-message">
          {{ successMessage }}
        </div>
      </div>
    </main>
  </div>
</template>
<style>
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
  font-family: 'Monsterrat';
  color: var(--text-color);
  background-color: var(--background-color);
  line-height: 1.6;
}

h1 {
  font-family: 'ogg-medium';
}

h2,
h3 {
  font-family: 'ogg-regular';
}

p {
  margin-bottom: 2.25rem;
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
  height: 63px;
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
  padding-right: var(--spacing-sm);
  padding-left: var(--spacing-sm);
  font-size: 1.3rem;
  line-height: 1.8;
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
  gap: 0.25rem;
  margin-bottom: 0.18rem;
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
  gap: var(--spacing-md);
}

.the-day-text {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.the-day-text p,
h1,
h2,
h3,
li,
ul {
  text-align: center;
  justify-content: center;
  align-items: center;
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

.location-header {
  margin-bottom: 6px;
}

.accommodations-text {
  justify-content: left;
  align-items: start;
  text-align: start;
  margin-top: 10px;
}

.day-list {
  margin-left: var(--spacing-md);
  list-style: none;
}

ul li {
  padding-top: 5px;
}

.day-list-header {
  margin-bottom: 1.25rem;
}

.timeline-text {
  justify-content: left;
  align-items: start;
  text-align: start;
}

.timeline-time,
.timeline-event {
  display: block;
  margin-bottom: 0.5rem;
}

.timeline-event {
  margin-bottom: 1.5rem;
}

.day-list li:last-child .timeline-event {
  margin-bottom: 0;
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

.attire-image {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-md);
}

.attire-image img {
  max-width: 100%;
  max-height: 80vh;
  object-fit: contain;
}

.attire-footbar {
  height: var(--spacing-lg);
  width: 100%;
  background-color: var(--secondary-color);
  margin-top: auto;
}

/* gift page specific designs */
.gift-page {
  align-content: center;
}

.registry-blurb {
  margin-top: var(--spacing-md);
}

.registry-header {
  margin-bottom: var(--spacing-md);
}

.success-message {
  margin-top: var(--spacing-sm);
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: var(--spacing-sm);
  margin-bottom: var(--spacing-sm);
}
th,
td {
  border: 1px solid #ccc;
  padding: 0.5em;
  text-align: left;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal-content {
  background: #fff;
  padding: 2em;
  border-radius: 8px;
  min-width: 300px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.2);
}

/* Responsive design */
@media (max-width: 768px) {

  .banner {
    background-image: url('/src/assets/banner-lift-mobile.jpg');
  }

  .nav-bar-shadow {
    height: 44px;
  }

  .landing-page .content-section+.content-section {
    padding-top: 0;
  }

  .landing-page .content-section:first-of-type {
    padding-bottom: 0;
  }

  .rsvp-form-image-section {
    flex-direction: column;
  }

  .rsvp-deadline-header {
    margin-top: 1rem;
    margin-bottom: 2rem;
    color: var(--primary-color);
    font-size: 10px;
  }

  .hidden-on-mobile {
    visibility: hidden !important;
    position: absolute !important;
    height: 0 !important;
    width: 0 !important;
  }

  .rsvp-image {
    order: -1;
    width: 100%;
    max-width: 100%;
    margin-bottom: var(--spacing-md);
  }

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
    font-size: 1.3rem;
    line-height: 1.8;
    margin-bottom: var(--spacing-md);
  }

  /* The day page mobile layout */
  .the-day-content {
    flex-direction: column;
  }

  .the-day-text,
  .location-text,
  .accommodations-text,
  .timeline-text,
  .food-drink-text,
  .day-list li {
    text-align: center;
    justify-content: center;
    align-items: center;
  }

  .attire-text,
  .attire-text p,
  .attire-text h2 {
    text-align: center !important;
    padding-left: var(--spacing-sm);
    padding-right: var(--spacing-sm);
  }

  .day-list {
    margin-left: 0;
    padding-left: 0;
    list-style: none;
  }

  .the-day-image img {
    max-width: 100%;
    margin-top: 20px;
  }

  /* Attire page adjustments */
  .attire-content {
    flex-direction: column;
  }

  .attire-image {
    order: 2;
    margin-top: var(--spacing-md);
    padding: var(--spacing-sm);
  }

  .attire-text {
    padding: var(--spacing-sm);
    order: 1;
  }

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
