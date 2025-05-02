<template>
    <div class="container">
      <div class="page-header mb-4">
        <h1>Favorites Report</h1>
        <p class="text-muted">View top favorited profiles and your favorites</p>
      </div>
      
      <ul class="nav nav-tabs mb-4" id="favoriteTabs" role="tablist">
        <li class="nav-item">
          <a 
            class="nav-link" 
            :class="{ active: activeTab === 'top' }" 
            id="top-tab" 
            @click.prevent="activeTab = 'top'" 
            href="#"
            role="tab"
          >
            Top 20 Favorited Profiles
          </a>
        </li>
        <li class="nav-item">
          <a 
            class="nav-link" 
            :class="{ active: activeTab === 'my' }" 
            id="my-tab" 
            @click.prevent="activeTab = 'my'" 
            href="#"
            role="tab"
          >
            My Favorites
          </a>
        </li>
      </ul>
      
      <div class="tab-content">
        <!-- Top Favorited Profiles Tab -->
        <div 
          class="tab-pane fade" 
          :class="{ 'show active': activeTab === 'top' }" 
          id="top" 
          role="tabpanel"
        >
          <div class="sort-controls mb-3">
            <div class="row">
              <div class="col-md-6">
                <h3>Top Favorited Profiles</h3>
              </div>
              <div class="col-md-6 text-right">
                <div class="form-inline justify-content-end">
                  <label for="topSortBy" class="mr-2">Sort by:</label>
                  <select id="topSortBy" v-model="topSortBy" class="form-control form-control-sm">
                    <option value="fav_count">Most Favorited</option>
                    <option value="name">Name</option>
                    <option value="parish">Parish</option>
                    <option value="age">Age</option>
                  </select>
                </div>
              </div>
            </div>
          </div>
          
          <div v-if="loadingTop" class="text-center my-5">
            <div class="spinner-border text-primary" role="status">
              <span class="sr-only">Loading...</span>
            </div>
          </div>
          
          <div v-else-if="errorTop" class="alert alert-danger">
            {{ errorTop }}
          </div>
          
          <div v-else-if="topFavorites.length === 0" class="alert alert-info">
            No favorited profiles found.
          </div>
          
          <div v-else class="row">
            <div v-for="profile in sortedTopFavorites" :key="profile.id" class="col-md-3 mb-4">
              <div class="card h-100">
                <img :src="profile.photo || '/static/images/default-profile.jpg'" class="card-img-top" alt="Profile photo">
                <div class="card-body">
                  <h5 class="card-title">{{ profile.name }}</h5>
                  <p class="card-text">
                    <small class="text-muted">{{ profile.age }} years • {{ profile.parish }}</small>
                  </p>
                  <p class="card-text">
                    <span class="badge badge-primary mr-1">{{ profile.fav_count }} favorites</span>
                  </p>
                  <router-link :to="`/profiles/${profile.id}`" class="btn btn-outline-primary btn-sm">
                    View Profile
                  </router-link>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- My Favorites Tab -->
        <div 
          class="tab-pane fade" 
          :class="{ 'show active': activeTab === 'my' }" 
          id="my" 
          role="tabpanel"
        >
          <div class="sort-controls mb-3">
            <div class="row">
              <div class="col-md-6">
                <h3>My Favorite Profiles</h3>
              </div>
              <div class="col-md-6 text-right">
                <div class="form-inline justify-content-end">
                  <label for="mySortBy" class="mr-2">Sort by:</label>
                  <select id="mySortBy" v-model="mySortBy" class="form-control form-control-sm">
                    <option value="name">Name</option>
                    <option value="parish">Parish</option>
                    <option value="age">Age</option>
                  </select>
                </div>
              </div>
            </div>
          </div>
          
          <div v-if="loadingMy" class="text-center my-5">
            <div class="spinner-border text-primary" role="status">
              <span class="sr-only">Loading...</span>
            </div>
          </div>
          
          <div v-else-if="errorMy" class="alert alert-danger">
            {{ errorMy }}
          </div>
          
          <div v-else-if="myFavorites.length === 0" class="alert alert-info">
            You haven't favorited any profiles yet. <router-link to="/">Explore profiles</router-link> to find your matches!
          </div>
          
          <div v-else class="row">
            <div v-for="profile in sortedMyFavorites" :key="profile.id" class="col-md-3 mb-4">
              <div class="card h-100">
                <img :src="profile.photo || '/static/images/default-profile.jpg'" class="card-img-top" alt="Profile photo">
                <div class="card-body">
                  <h5 class="card-title">{{ profile.name }}</h5>
                  <p class="card-text">
                    <small class="text-muted">{{ profile.age }} years • {{ profile.parish }}</small>
                  </p>
                  <div class="d-flex justify-content-between align-items-center">
                    <router-link :to="`/profiles/${profile.id}`" class="btn btn-outline-primary btn-sm">
                      View Profile
                    </router-link>
                    <button @click="removeFavorite(profile.id)" class="btn btn-outline-danger btn-sm">
                      <i class="fas fa-heart"></i> Remove
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { mapGetters } from 'vuex';
  import axios from 'axios';
  
  export default {
    name: 'FavoritesReport',
    data() {
      return {
        activeTab: 'top',
        topSortBy: 'fav_count',
        mySortBy: 'name',
        topFavorites: [],
        myFavorites: [],
        loadingTop: true,
        loadingMy: true,
        errorTop: null,
        errorMy: null
      };
    },
    computed: {
      ...mapGetters(['isAuthenticated', 'currentUser', 'getToken']),
      
      sortedTopFavorites() {
        return this.sortProfiles(this.topFavorites, this.topSortBy);
      },
      
      sortedMyFavorites() {
        return this.sortProfiles(this.myFavorites, this.mySortBy);
      }
    },
    methods: {
      fetchTopFavorites() {
        this.loadingTop = true;
        this.errorTop = null;
        
        axios.get('/api/users/favourties/20', {
          headers: { Authorization: `Bearer ${this.getToken}` }
        })
          .then(response => {
            this.topFavorites = response.data.favorites;
            this.loadingTop = false;
          })
          .catch(error => {
            console.error('Error fetching top favorites:', error);
            this.errorTop = 'Failed to load top favorited profiles. Please try again.';
            this.loadingTop = false;
          });
      },
      
      fetchMyFavorites() {
        if (!this.isAuthenticated || !this.currentUser) {
          this.errorMy = 'You must be logged in to view your favorites.';
          this.loadingMy = false;
          return;
        }
        
        this.loadingMy = true;
        this.errorMy = null;
        
        axios.get(`/api/users/${this.currentUser.id}/favourites`, {
          headers: { Authorization: `Bearer ${this.getToken}` }
        })
          .then(response => {
            this.myFavorites = response.data.favorites;
            this.loadingMy = false;
          })
          .catch(error => {
            console.error('Error fetching user favorites:', error);
            this.errorMy = 'Failed to load your favorites. Please try again.';
            this.loadingMy = false;
          });
      },
      
      removeFavorite(profileId) {
        if (!confirm('Are you sure you want to remove this profile from your favorites?')) {
          return;
        }
        
        axios.delete(`/api/profiles/${profileId}/favourite`, {
          headers: { Authorization: `Bearer ${this.getToken}` }
        })
          .then(() => {
            // Remove from local array
            this.myFavorites = this.myFavorites.filter(profile => profile.id !== profileId);
            this.$toasted.success('Profile removed from favorites');
          })
          .catch(error => {
            console.error('Error removing favorite:', error);
            this.$toasted.error('Failed to remove favorite. Please try again.');
          });
      },
      
      sortProfiles(profiles, sortBy) {
        const sorted = [...profiles];
        
        switch (sortBy) {
          case 'name':
            sorted.sort((a, b) => a.name.localeCompare(b.name));
            break;
            
          case 'parish':
            sorted.sort((a, b) => a.parish.localeCompare(b.parish));
            break;
            
          case 'age':
            sorted.sort((a, b) => b.age - a.age);
            break;
            
          case 'fav_count':
            sorted.sort((a, b) => b.fav_count - a.fav_count);
            break;
            
          default:
            // Default to sorting by name
            sorted.sort((a, b) => a.name.localeCompare(b.name));
        }
        
        return sorted;
      },
      
      refreshData() {
        if (this.activeTab === 'top') {
          this.fetchTopFavorites();
        } else {
          this.fetchMyFavorites();
        }
      }
    },
    watch: {
      activeTab() {
        this.refreshData();
      }
    },
    created() {
      if (!this.isAuthenticated) {
        this.$router.push('/login');
        return;
      }
      
      // Initial data fetch based on active tab
      this.refreshData();
    }
  };
  </script>
  
  <style scoped>
  .card {
    transition: transform 0.2s;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
  
  .card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
  }
  
  .card-img-top {
    height: 200px;
    object-fit: cover;
  }
  
  .tab-pane {
    padding-top: 1rem;
  }
  
  .badge {
    font-size: 0.8rem;
  }
  
  .text-right {
    text-align: right;
  }
  </style>