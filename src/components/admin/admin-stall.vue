<template>
  <!-- Added Navigation Bar -->
  <nav>
    <ul class="sidebar" ref="sidebar">
      <li @click="hideSidebar">
        <a href="#">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="26"
            viewBox="0 -960 960 960"
            width="26"
            fill="#e3e3e3"
          >
            <path
              d="m256-200-56-56 224-224-224-224 56-56 224 224 224-224 56 56-224 224 224 224-56 56-224-224-224 224Z"
            />
          </svg>
        </a>
      </li>
      <li><a href="#" @click.prevent="$router.push('/')">Home</a></li>
      <li><a href="#" @click.prevent="logout">Log Out</a></li>
    </ul>

    <ul>
      <li>
        <a href="#" @click.prevent="$router.push('/')"
          >Market Price Tracker-Admin</a
        >
      </li>
      <li class="hideMobile">
        <a href="#" @click.prevent="logout">Log Out</a>
      </li>
      <li class="menu-btn" @click="showSidebar">
        <a href="#">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="26"
            viewBox="0 -960 960 960"
            width="26"
            fill="#e3e3e3"
          >
            <path
              d="M120-240v-80h720v80H120Zm0-200v-80h720v80H120Zm0-200v-80h720v80H120Z"
            />
          </svg>
        </a>
      </li>
    </ul>
  </nav>

  <h1 class="top-text">Admin Price</h1>

  <div class="image-container">
    <img src="@/assets/main.jpeg" class="main-image" alt="Blurred Background" />
    <div class="img-overlay"></div>
  </div>

  <div class="main-content">
    <div class="addItem">
      <div class="form-box">
        <form @submit.prevent="editItem">
          <br />
          <p>Enter new price:</p>
          <br />

          <select 
            v-model="selectedItem" 
            class="btn" 
            required
            :disabled="isLoading"
          >
            <option value="">-- Select an item --</option>
            <option 
              v-for="price in prices" 
              :key="price.RiceType" 
              :value="price.RiceType"
            >
              {{ price.RiceType }}
            </option>
          </select>
          <br />

          <input
            type="number"
            step="0.01"
            class="input-field"
            placeholder="Price"
            v-model.number="FormDataP.newPrice"
            required
            min="0.01"
            :disabled="isLoading"
          />
          <br />

          <button 
            type="submit" 
            class="btn"
            :disabled="isLoading || !selectedItem || !FormDataP.newPrice"
          >
            {{ isLoading ? 'Processing...' : 'Change' }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { removeToken, getToken } from "@/utils/auth";

export default {
  name: "adminStall",
  data() {
    const userData = JSON.parse(localStorage.getItem('userData') || '{}');
    return {
      urlappphp: "https://star-panda-literally.ngrok-free.app/app.php",
      FormDataP: {
        newPrice: "",
      },
      prices: [],
      selectedItem: "",
      isLoading: false,
      market: userData.market || '',
      selectedSeat: userData.stall || '',
      category: 'rice' // default category
    };
  },

  methods: {
    showSidebar() {
  this.$refs.sidebar.style.display = "flex";
},

hideSidebar() {
  this.$refs.sidebar.style.display = "none";
},
    async editItem() {
      try {
        if (!this.selectedItem || !this.FormDataP.newPrice) {
          alert("Please select an item and enter a price.");
          return;
        }

        this.isLoading = true;
        const token = getToken();
        if (!token) {
          alert("Your session has expired. Please login again.");
          this.$router.replace("/login");
          return;
        }

        const userData = JSON.parse(localStorage.getItem('userData') || '{}');
        if (!userData.market || !userData.stall) {
          alert("Missing market or stall information. Please login again.");
          this.$router.replace("/login");
          return;
        }

        const response = await fetch(this.urlappphp, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${token}`,
          },
          body: JSON.stringify({
            action: "changePrice",
            itemName: this.selectedItem,
            newPrice: parseFloat(this.FormDataP.newPrice),
            market: userData.market,
            stall_no: userData.stall,
            sourceUrl: "/addedadmin"
          }),
        });

        const result = await response.json();

        if (result.success) {
          alert("Price updated successfully!");
          this.FormDataP.newPrice = "";
          this.selectedItem = "";
          await this.getPrices(); // Refresh the price list
        } else {
          alert(result.error || "Failed to update price");
        }
      } catch (error) {
        console.error("Error updating price:", error);
        alert("An error occurred while updating the price");
      } finally {
        this.isLoading = false;
      }
    },
    async logout() {
      try {
        removeToken();
        this.localUserData = {};
        this.$router.replace("/");
      } catch (error) {
        console.error("Logout error:", error);
      }
    },

    async getPrices(marketName, category, seat) {
      this.market = marketName || this.market;
      this.category = category || this.category;
      this.selectedSeat = seat || this.selectedSeat;

      const urls = [
        "https://star-panda-literally.ngrok-free.app/app.php",
        "https://rusiann7.helioho.st/app.php",
        "http://localhost:8080/app.php",
      ];

      for (const url of urls) {
        try {
          const testResponse = await fetch(url, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ action: "ping" }),
          });

          if (testResponse.ok) {
            this.urlappphp = url;
            break;
          }
        } catch (_) {
          continue;
        }
      }

      try {
        const response = await fetch(this.urlappphp, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            action: "getPrices",
            market: this.market,
            category: this.category.toLowerCase(),
            seat: this.selectedSeat,
          }),
        });

        const result = await response.json();

        if (result.success) {
          this.prices = result.data.map((item) => ({
            ...item,
            Price:
              typeof item.Price === "number"
                ? item.Price.toFixed(2)
                : item.Price,
          }));
          this.filteredPrices = [...this.prices];
        } else {
          console.error("Failed to fetch Prices:", result.error);
        }
      } catch (error) {
        console.error("Error fetching Prices:", error);
      } finally {
        this.isLoading = false;
      }
    },
  },
  mounted() {
    // Get user data from localStorage
    const userData = JSON.parse(localStorage.getItem('userData') || '{}');
      
    if (!userData.market || !userData.stall) {
      this.$router.replace('/login');
      return;
    }
      
    // Set initial data
    this.market = userData.market;
    this.selectedSeat = userData.stall;
      
    // Initial price fetch
    this.getPrices();
      
    // Set up auto-refresh
    this.priceInterval = setInterval(() => {
      this.getPrices();
    }, 5000);
  },

    beforeUnmount() {
      if (this.priceInterval) {
        clearInterval(this.priceInterval);
      }
    },
};
</script>

<style scoped>
/* Added navbar styles */
* {
  margin: 0;
  padding: 0;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  padding: 0;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

nav {
  background-color: #2d333f;
  box-shadow: 3px 3px 5px rgba(0, 0, 0, 0.1);
  position: fixed;
  top: 0;
  z-index: 100;
  margin: 0;
  padding: 0;
  width: 100%;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

nav ul {
  width: 100%;
  list-style: none;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin: 0;
  padding: 0;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

nav li {
  height: 50px;
  margin: 0;
  padding: 0;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

nav a {
  height: 100%;
  padding: 0 30px;
  text-decoration: none;
  display: flex;
  align-items: center;
  color: #e3e3e3;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  font-weight: 500;
  font-size: 15px;
  letter-spacing: 0.5px;
}

nav li:first-child a {
  font-size: 18px;
  font-weight: 600;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

nav a:hover {
  background-color: #3a4252;
  color: white;
}

nav a:active {
  background-color: #4a5568; /* pressed state */
}

nav li:first-child {
  margin-right: auto;
}

.sidebar {
  position: fixed;
  top: 0;
  right: 0;
  height: 100vh;
  width: 250px;
  z-index: 999;
  background-color: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  box-shadow: -10px 0 10px rgba(0, 0, 0, 0.1);
  display: none;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  margin: 0;
  padding: 0;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.sidebar li {
  width: 100%;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.sidebar a {
  width: 100%;
  font-size: 16px;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.sidebar a:hover {
  background-color: #3a4252;
}

.menu-btn {
  display: none;
}

.menu-btn:hover {
  background-color: #3a4252;
}

@media (max-width: 800px) {
  .hideMobile {
    display: none;
  }
  .menu-btn {
    display: block;
  }
}

@media (max-width: 400px) {
  .sidebar {
    width: 100%;
  }
}

.clickable-cell {
  cursor: pointer;
  padding: 12px 15px; /* table cell padding */
}

.header-section {
  display: flex;
  align-items: center; /* vertical alignment */
  justify-content: space-between; /* put title left, button right */
  margin-bottom: 24px;
  gap: 16px;
  padding: 8px 0;
  color: #ffffff;
}

.section-title {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 600;
  color: #ffffff;
}

.btn {
  margin-top: 10px;
  padding: 10px 20px;
  background: #ffe082;
  color: #001821;
  border: none;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.btn:hover {
  background: #ffd448;
  color: #001821;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.btn-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
  border-radius: 4px;
}

.btn svg {
  fill: #000000;
}

.list-price-container {
  width: 100%;
  overflow: hidden;
  background-color: #232831;
  border-radius: 15px; /* rounded corners */
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  margin: 15px auto; /* center   */
  padding: 15px;
}

.price-list {
  width: 100%;
  overflow: hidden;
}

.scroll {
  max-height: 500px;
  overflow-y: auto;
  position: relative;
  border-radius: 4px; /* rounded corners */
}

.main-content {
  position: relative; /* above overlay */
  z-index: 2;
  padding-top: 0;
  margin-top: -180px;
  align-items: center;
  justify-content: center;
  display: flex;
  min-height: calc(100vh - 50px);
  width: 100%;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  padding-left: 15px;
  padding-right: 15px;
}

.table-content {
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 0.9rem;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  min-width: 900px;
  width: 100%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  border-radius: 15px;
}

.table-content thead tr {
  background-color: #1e1e1e;
  color: #ffffff;
  text-align: left;
  font-weight: bold;
}

.table-content tbody tr {
  background-color: #f9f9f9;
  color: #333;
  border-bottom: 1px solid #dddddd;
}

.table-content tbody tr:nth-of-type(even) {
  background-color: #f3f3f3;
}

.table-content tbody tr:hover {
  background-color: #e9e9e9; /* hover effect */
  cursor: pointer;
}

.selected-row {
  background-color: #ffe082 !important; /* selected row */
  color: #333 !important;
}

.table-content th,
.table-content td {
  padding: 12px 15px;
}

.table-content tbody tr:last-of-type {
  border-bottom: #1e1e1e;
}

@media (max-width: 768px) {
  .table-content {
    min-width: 100%;
    font-size: 0.85rem;
  }

  .table-content th,
  .table-content td {
    padding: 10px 12px;
  }

  .main-content {
    padding: 0 10px;
  }
}

@media (max-width: 480px) {
  .table-content {
    font-size: 0.8rem;
  }

  .table-content th,
  .table-content td {
    padding: 8px 10px;
  }

  .compare-content {
    padding: 15px;
  }
}

.addItem {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 20px;
  margin: 10px;
  max-width: 400px;
  width: 90%;
  color: white;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background-color: #232831;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  box-sizing: border-box;
  min-height: auto;
  max-height: 475px;
  align-self: center;
  margin-top: 125px;
  margin-bottom: 0;
  font-size: 17px;
}

.form-box {
  width: 100%;
}

.input-field {
  margin-bottom: 15px; /* space below each input field */
  margin-top: 15px;
  padding: 12px 15px;
  width: 100%;
  font-size: 16px;
  border-radius: 6px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}

.top-text {
  position: relative; /* above overlay */
  z-index: 2;
  color: #ffffff;
  text-align: center;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  padding-top: 100px;
  font-size: 2rem;
  margin-bottom: 20px;
}

.loading-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  color: white;
}

.loading-spinner {
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top: 4px solid #ffffff;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.image-container {
  position: fixed;
  width: 100%;
  height: 100vh;
  top: 0;
  left: 0;
  z-index: 1;
}

.main-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: blur(5px);
  position: absolute;
  top: 0;
  left: 0;
}

.img-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(45, 51, 63, 0.452);
}

.header-section {
  display: flex;
  align-items: center; /* vertical alignment */
  justify-content: space-between; /* put title left, button right */
  margin-bottom: 24px;
  gap: 16px;
  padding: 8px 0;
  color: #ffffff;
}

.tab {
  padding: 0.5rem 1rem;
  background: #ffe082;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  color: #001821;
}

.tab.active {
  background: #333;
  color: #fff;
}
</style>
