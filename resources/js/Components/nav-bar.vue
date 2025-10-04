<script setup>
import { layoutMethods } from "@/state/helpers";
import { Link, router } from '@inertiajs/vue3';
import simplebar from "simplebar-vue";
import { ref, onMounted, computed } from 'vue';
import i18n from '../i18n';
import { initI18n, loadLocaleMessages } from '../i18n';
import { useSharedState } from '@/composables/useSharedState';
import store from '/resources/js/state/store.js';
import axios from 'axios';
import { formatDistanceToNowStrict } from 'date-fns';
import Swal from "sweetalert2";

// Existing state
const selectedLanguageCode = ref(i18n.global.locale);
const {
  languages,
  fetchData,
  locations,
  fetchLocations,
  selectedLocation,
  setServiceLocation,
  notifications,
  chats,
  unreadChat,
  unreadNotification,
  fetchFirebaseSettings,
  handleNotificationClick,
  handleMarkAllAsRead
} = useSharedState();

// Search functionality
const searchQuery = ref('');
const showSearchResults = ref(false);
const searchInputRef = ref(null);

// Define all menu items with their routes and permissions
const allMenuItems = ref([
  // Dashboard
  { name: 'Dashboard', route: '/dashboard', permission: 'access-home', icon: 'ri-home-4-line' },
  { name: 'Chat', route: '/chat', permission: 'chat', icon: 'bx bx-message-rounded-dots' },
  
  // Order Management - Promotion
  { name: 'Promo Code', route: '/promo-code', permission: 'manage-promo', icon: 'ri-gift-line', parent: 'Promotion Management' },
  { name: 'Send Notification', route: '/push-notifications', permission: 'view-notifications', icon: 'ri-gift-line', parent: 'Promotion Management' },
  { name: 'Banner Image', route: '/banner-image', permission: 'banner_image', icon: 'ri-gift-line', parent: 'Promotion Management' },
  
  // Order Management - Price
  { name: 'Service Location', route: '/service-locations', permission: 'service-location', icon: 'bx bx-money', parent: 'Price Management' },
  { name: 'Zone', route: '/zones', permission: 'view-zone', icon: 'bx bx-money', parent: 'Price Management' },
  { name: 'Airport', route: '/airport', permission: 'view-airport', icon: 'bx bx-money', parent: 'Price Management' },
  { name: 'Vehicle Type', route: '/vehicle_type', permission: 'vehicle-types', icon: 'bx bx-money', parent: 'Price Management' },
  { name: 'Rental Package', route: '/rental-package-types', permission: 'rental-package', icon: 'bx bx-money', parent: 'Price Management' },
  { name: 'Set Price', route: '/set-prices', permission: 'vehicle-fare', icon: 'bx bx-money', parent: 'Price Management' },
  { name: 'Goods Type', route: '/goods-type', permission: 'manage-goods-types', icon: 'bx bx-money', parent: 'Price Management' },
  
  // Order Management - Geofencing
  { name: 'Heat Map', route: '/map/heat_map', permission: 'heat-map', icon: 'bx bx-map', parent: 'Geofencing' },
  { name: 'Gods Eye', route: '/map/gods_eye', permission: 'gods-eye', icon: 'bx bx-map', parent: 'Geofencing' },
  { name: 'Peak Zone', route: '/peak_zone', permission: 'peak-zone-view', icon: 'bx bx-map', parent: 'Geofencing' },
  
  // Order Management - Others
  { name: 'Delivery Requests', route: '/delivery-rides-request', permission: 'manage-delivery-request', icon: 'bx bx-package' },
  { name: 'Ongoing Rides', route: '/ongoing-rides', permission: 'ongoing-request-view', icon: 'ri-taxi-fill' },
  { name: 'SOS', route: '/sos', permission: 'view-sos', icon: 'mdi mdi-dots-horizontal-circle', parent: 'Others' },
  { name: 'Cancellation', route: '/cancellation', permission: 'view-cancellation', icon: 'mdi mdi-dots-horizontal-circle', parent: 'Others' },
  { name: 'FAQ', route: '/faq', permission: 'view-faq', icon: 'mdi mdi-dots-horizontal-circle', parent: 'Others' },
  
  // Users - Customer Management
  { name: 'User List', route: '/users', permission: 'view-users', icon: 'ri-team-fill', parent: 'Customer Management' },
  { name: 'Delete Request Users', route: '/users/deleted-user', permission: 'view-delete-user', icon: 'ri-team-fill', parent: 'Customer Management' },
  { name: 'User Bulk Upload', route: '/user-import', permission: 'bulk_upload', icon: 'ri-team-fill', parent: 'Customer Management' },
  
  // Users - Driver Management
  { name: 'Pending Drivers', route: '/pending-drivers', permission: 'view-approval-pending-drivers', icon: 'ri-user-follow-line', parent: 'Driver Management' },
  { name: 'Approved Drivers', route: '/approved-drivers', permission: 'view-approved-drivers', icon: 'ri-user-follow-line', parent: 'Driver Management' },
  { name: 'Subscription', route: '/subscription', permission: 'manage-subscription', icon: 'ri-user-follow-line', parent: 'Driver Management' },
  { name: 'Drivers Ratings', route: '/drivers-rating', permission: 'driver-rating-list', icon: 'ri-user-follow-line', parent: 'Driver Management' },
  
  // Users - Admin Management
  { name: 'Admins', route: '/admins', permission: 'admin', icon: 'ri-group-line', parent: 'Admin Management' },
  
  // Users - Others
  { name: 'Referral Settings', route: '/referral-settings', permission: 'referral-settings-view', icon: 'bx bx-share-alt' },
  { name: 'Ticket Title', route: '/title', permission: 'view-ticket-title', icon: 'bx bx-support', parent: 'Support Management' },
  { name: 'Support Tickets', route: '/support-tickets', permission: 'view-support-ticket', icon: 'bx bx-support', parent: 'Support Management' },
  
  // Users - Reports
  { name: 'User Report', route: '/report/user-report', permission: 'user-report', icon: 'ri-file-3-fill', parent: 'Report' },
  { name: 'Driver Report', route: '/report/driver-report', permission: 'driver-report', icon: 'ri-file-3-fill', parent: 'Report' },
  
  // Settings - Business Settings
  { name: 'General Settings', route: '/general-settings', permission: 'general-settings-view', icon: 'ri-settings-5-fill', parent: 'Business Settings' },
  { name: 'Customization Settings', route: '/customization-settings', permission: 'customization-settings-view', icon: 'ri-settings-5-fill', parent: 'Business Settings' },
  
  // Settings - App Settings
  { name: 'Wallet Settings', route: '/wallet-settings', permission: 'wallet-settings-view', icon: 'mdi mdi-cellphone-cog', parent: 'App Settings' },
  { name: 'Tip Settings', route: '/tip-settings', permission: 'tip-settings-view', icon: 'mdi mdi-cellphone-cog', parent: 'App Settings' },
  
  // Settings - Third Party Settings
  { name: 'Payment Gateway Settings', route: '/payment-gateway', permission: 'payment-gateway-settings-view', icon: 'mdi mdi-cogs', parent: 'Third Party Settings' },
  { name: 'Firebase Settings', route: '/firebase', permission: 'firebase-settings-view', icon: 'mdi mdi-cogs', parent: 'Third Party Settings' },
]);

// Computed property to filter menu items based on search and permissions
const filteredMenuItems = computed(() => {
  if (!searchQuery.value) return [];
  
  const query = searchQuery.value.toLowerCase();
  const userPermissions = store.getters.permissions || [];
  
  return allMenuItems.value.filter(item => {
    const hasPermission = !item.permission || userPermissions.includes(item.permission);
    const matchesSearch = item.name.toLowerCase().includes(query) || 
                         (item.parent && item.parent.toLowerCase().includes(query));
    return hasPermission && matchesSearch;
  }).slice(0, 8);
});

// Navigate to selected page
const navigateToPage = (route) => {
  router.get(route);
  searchQuery.value = '';
  showSearchResults.value = false;
};

// Handle click outside to close search results
const handleClickOutside = (event) => {
  if (searchInputRef.value && !searchInputRef.value.contains(event.target)) {
    showSearchResults.value = false;
  }
};

const formatTimestamp = (timestamp) => {
  const date = new Date(timestamp);
  if (isNaN(date.getTime())) {
    console.error("Invalid timestamp:", timestamp);
    return "Invalid date";
  }
  return formatDistanceToNowStrict(date, { addSuffix: true });
};

const selectedLabel = ref('');

onMounted(async () => {
  store.dispatch('fetchPermissions');
  await fetchData();
  await fetchLocations();
  await fetchFirebaseSettings();
  
  document.addEventListener('click', handleClickOutside);
  
  var currentLocation = localStorage.getItem('selectedLocation');
  if(!currentLocation){
    currentLocation = locations.value?.[0]?.value;
    localStorage.setItem('selectedLocation', currentLocation); 
    setServiceLocation(currentLocation);
  }

  const serviceLocation = locations.value.find(l => l.value === currentLocation); 
  selectedLabel.value = serviceLocation?.label
 
  const currentLocale = localStorage.getItem('locale') || defaultLocale;
  selectedLanguageCode.value = currentLocale;
  const selectedLanguage = languages.value.find(lang => lang.code === currentLocale);

  if (selectedLanguage) {
    const direction = selectedLanguage.direction;
    const body = document.body;

    if (direction === 'rtl') {
      localStorage.setItem('toggleDirection', true);
      body.classList.add('rtl');
      body.classList.remove('ltr');
    } else {
      localStorage.setItem('toggleDirection', false);
      body.classList.add('ltr');
      body.classList.remove('rtl');
    }
  }
});

const setLocation = async (location) => {
  setServiceLocation(location);
  const currentLocation = localStorage.getItem('selectedLocation');
  if(currentLocation){
    const match = locations.value.find(l => l.value === currentLocation);
    selectedLabel.value = match ? match.label : '';
    window.location.reload();
    localStorage.setItem('selectedLocation', match.value);
  }
}

const setLanguage = async (locale) => {
  await loadLocaleMessages(locale);
  i18n.global.locale = locale;
  selectedLanguageCode.value = locale;
  localStorage.setItem('locale', locale);

  const selectedLanguage = languages.value.find(lang => lang.code === locale);

  if (selectedLanguage) {
    const direction = selectedLanguage.direction;
    const body = document.body;

    if (direction === 'rtl') {      
      window.location.reload();
      localStorage.setItem('toggleDirection', true);
      body.classList.add('rtl');
      body.classList.remove('ltr');
    } else {
      window.location.reload();
      localStorage.setItem('toggleDirection', false);
      body.classList.add('ltr');
      body.classList.remove('rtl');
    }
  }
};

const handleChatClick = (chat_id) => {
  router.get('/chat?conversation_id='+ chat_id);
}

const logout = () => {
  Swal.fire({
    title: i18n.global.t("are_you_sure"),
    text: i18n.global.t("you_want_to_be_logout"),
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#34c38f",
    cancelButtonColor: "#f46a6a",
    confirmButtonText: i18n.global.t("yes"),
    cancelButtonText: i18n.global.t("cancel"),
  }).then((result) => {
    if (result.isConfirmed) {
      router.post(route("logout"));
    }
  });
};
</script>

<script>
import { ref, onMounted } from 'vue';
import simplebar from "simplebar-vue";
import i18n from "../i18n";
import { Link } from '@inertiajs/vue3';
import { mapGetters } from 'vuex';

export default {
  data() {
    return {
      lan: i18n.global.locale,
      text: null,
      flag: null,
      value: null,
      myVar: 1, 
      profilePhotoUrl: this.$page.props.auth.user.profile_picture,       
    };
  },
  components: {
    simplebar,
    Link,
  },
  methods: {
    ...layoutMethods,
    
    toggleHamburgerMenu() {
      const windowSize = document.documentElement.clientWidth;
      const layoutType = document.documentElement.getAttribute("data-layout");

      document.documentElement.setAttribute("data-sidebar-visibility", "show");
      const visibilityType = document.documentElement.getAttribute("data-sidebar-visibility");

      if (windowSize > 767) document.querySelector(".hamburger-icon").classList.toggle("open");

      if (layoutType === "horizontal") {
        document.body.classList.toggle("menu");
      }

      if (visibilityType === "show" && (layoutType === "vertical" || layoutType === "semibox")) {
        if (windowSize < 1025 && windowSize > 767) {
          document.body.classList.remove("vertical-sidebar-enable");
          document.documentElement.setAttribute("data-sidebar-size", document.documentElement.getAttribute("data-sidebar-size") === "sm" ? "" : "sm");
        } else if (windowSize > 1025) {
          document.body.classList.remove("vertical-sidebar-enable");
          document.documentElement.setAttribute("data-sidebar-size", document.documentElement.getAttribute("data-sidebar-size") === "lg" ? "sm" : "lg");
        } else if (windowSize <= 767) {
          document.body.classList.add("vertical-sidebar-enable");
          document.documentElement.setAttribute("data-sidebar-size", "lg");
        }
      }

      if (layoutType === "twocolumn") {
        document.body.classList.toggle("twocolumn-panel");
      }
    },
    
    toggleMenu() {
      this.$parent.toggleMenu();
    },
    
    toggleRightSidebar() {
      this.$parent.toggleRightSidebar();
    },
    
    initFullScreen() {
      document.body.classList.toggle("fullscreen-enable");
      if (!document.fullscreenElement && !document.mozFullScreenElement && !document.webkitFullscreenElement) {
        if (document.documentElement.requestFullscreen) {
          document.documentElement.requestFullscreen();
        } else if (document.documentElement.mozRequestFullScreen) {
          document.documentElement.mozRequestFullScreen();
        } else if (document.documentElement.webkitRequestFullscreen) {
          document.documentElement.webkitRequestFullscreen(Element.ALLOW_KEYBOARD_INPUT);
        }
      } else {
        if (document.cancelFullScreen) {
          document.cancelFullScreen();
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen();
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen();
        }
      }
    },
    
    toggleDarkMode() {
      const theme = document.documentElement.getAttribute("data-bs-theme") === "dark" ? "light" : "dark";
      const sidebarColor = document.documentElement.getAttribute("data-sidebar") === "dark" ? "light" : "dark";
      document.documentElement.setAttribute("data-bs-theme", theme);
      document.documentElement.setAttribute("data-sidebar", sidebarColor);     

      localStorage.setItem('toggleDarkMode', theme === 'dark');  

      this.changeMode({ mode: theme });
      this.changeSidebarColor({ sidebarColor: sidebarColor });
    },
    
    savedToggleTheme() {
      const isDarkMode = localStorage.getItem('toggleDarkMode') === 'true';
      const theme = isDarkMode ? 'dark' : 'light';
      const sidebarColor = isDarkMode ? 'dark' : 'light';

      document.documentElement.setAttribute("data-bs-theme", theme);
      document.documentElement.setAttribute("data-sidebar", sidebarColor);      
      this.changeMode({ mode: theme });
      this.changeSidebarColor({ sidebarColor: sidebarColor });
    },
    
    removeItem(cartItem) {
      this.cartItems = this.cartItems.filter(item => item.id !== cartItem.id);
      this.$emit("cart-item-price", this.cartItems.length);
    },
  },
  computed: {
    ...mapGetters(['permissions']),
    
    calculateTotalPrice() {
      return this.cartItems.reduce((total, item) => total + parseFloat(item.itemPrice), 0).toFixed(2);
    },
    
    supportTicket() {
      return window.supportTicket || 0;
    }
  },
  mounted() {
    this.savedToggleTheme();
    this.flag = this.$i18n.locale;
    document.addEventListener("scroll", function () {
      const pageTopbar = document.getElementById("page-topbar");
      if (pageTopbar) {
        document.body.scrollTop >= 50 || document.documentElement.scrollTop >= 50
          ? pageTopbar.classList.add("topbar-shadow")
          : pageTopbar.classList.remove("topbar-shadow");
      }
    });
    if (document.getElementById("topnav-hamburger-icon")) {
      document.getElementById("topnav-hamburger-icon").addEventListener("click", this.toggleHamburgerMenu);
    }
  },
};
</script>

<template>
  <header id="page-topbar">
      <div class="layout-width">
          <div class="d-flex align-items-center justify-content-between">  
              <div>
                  <nav class="navbar navbar-expand">
                      <div class="top-navbar">
                          <div class="navbar-nav nav">
                              <ul class="navbar-nav">
                                <li class="nav-item dropdown" v-if="permissions.includes('access-home')">
                                      <Link class="nav-link" href="/dashboard" @click.prevent
                                          :class="{ 'active': ['/dashboard'].some(path => $page.url.startsWith(path)) }">
                                          <i class="ri-home-4-line"></i>
                                          <span class="ms-2">Dashboard</span>
                                      </Link> 
                                </li>
                                <li  class="nav-item dropdown"  v-if="permissions.includes('chat')">
                                              <Link class="nav-link dropdown-item" href="/chat" @click.prevent
                                              :class="{ 'active': ['/chat'].some(path => $page.url.startsWith(path)) }">
                                                  <i class="bx bx-message-rounded-dots"></i>{{ $t("chat") }}
                                              </Link>
                                  </li>
                                  <!-- Order Management Menu -->
                                  <li class="nav-item dropdown" v-if="permissions.includes('access-home')">
                                      <a class="nav-link" href="#" @click.prevent
                                          :class="{ 'active': ['/promo-code','/set-prices', '/drivers-levelup','/incentives',
                                          '/rental-package-types','/dispatch/delivery-package-ride','/dispatch/delivery-ride','/airport',
                                          '/goods-type','/dispatch/delivery-ride','/service-locations','/zones' ,'/vehicle_type','/map/heat_map',
                                          '/map/gods_eye','/ongoing-rides'].some(path => $page.url.startsWith(path)) }">
                                          <i class="ri-home-4-line"></i>
                                          <span class="ms-2">Order management</span>
                                      </a> 
                                      <ul class="dropdown-menu-mega">                          
                                          
                                          
                                          <!-- Promotion Management -->
                                          <li class="dropdown-submenu" v-if="permissions.includes('promotion-management')">
                                              <a class="dropdown-item" href="#" @click.prevent>
                                                  <i class="ri-gift-line"></i>{{ $t("promotion-management") }}
                                              </a>
                                              <ul class="dropdown-menu">
                                                  <li v-if="permissions.includes('manage-promo')">
                                                      <Link class="dropdown-item" href="/promo-code">{{ $t("promo_code") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('view-notifications')">
                                                      <Link class="dropdown-item" href="/push-notifications">{{ $t("send-notification") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('banner_image')">
                                                      <Link class="dropdown-item" href="/banner-image">{{ $t("banner-image") }}</Link>
                                                  </li>
                                              </ul>
                                          </li>
                                          
                                          <!-- Price Management -->
                                          <li class="dropdown-submenu" v-if="permissions.includes('price-management')">
                                              <a class="dropdown-item" href="#" @click.prevent>
                                                  <i class="bx bx-money"></i>{{ $t("price-management") }}
                                              </a>
                                              <ul class="dropdown-menu">
                                                  <li v-if="permissions.includes('service-location')">
                                                      <Link class="dropdown-item" href="/service-locations">{{ $t("service_location") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('view-zone')">
                                                      <Link class="dropdown-item" href="/zones">{{ $t("zone") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('view-airport')">
                                                      <Link class="dropdown-item" href="/airport">{{ $t("airport") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('vehicle-types')">
                                                      <Link class="dropdown-item" href="/vehicle_type">{{ $t("vehicle_type") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('rental-package')">
                                                      <Link class="dropdown-item" href="/rental-package-types">{{ $t("rental-package") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('vehicle-fare')">
                                                      <Link class="dropdown-item" href="/set-prices">{{ $t("set-price") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('manage-goods-types')">
                                                      <Link class="dropdown-item" href="/goods-type">{{ $t("goods-type") }}</Link>
                                                  </li>
                                              </ul>
                                          </li>
                                          
                                          <!-- Geofencing -->
                                          <li class="dropdown-submenu" v-if="permissions.includes('geo-fencing')">
                                              <a class="dropdown-item" href="#" @click.prevent>
                                                  <i class="bx bx-map"></i>{{ $t("geofencing") }}
                                              </a>
                                              <ul class="dropdown-menu">
                                                  <li v-if="permissions.includes('heat-map')">
                                                      <Link class="dropdown-item" href="/map/heat_map">{{ $t("heat-map") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('gods-eye')">
                                                      <Link class="dropdown-item" href="/map/gods_eye">{{ $t("gods-eye") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('peak-zone-view')">
                                                      <Link class="dropdown-item" href="/peak_zone">{{ $t("peakzone") }}</Link>
                                                  </li>
                                              </ul>
                                          </li>
                                          
                                          <!-- Delivery Requests -->
                                          <li v-if="permissions.includes('manage-delivery-request')">
                                              <Link class="dropdown-item" href="/delivery-rides-request">
                                                  <i class="bx bx-package"></i>{{ $t("delivery-requests") }}
                                              </Link>
                                          </li>
                                          
                                          <!-- Ongoing Rides -->
                                          <li v-if="permissions.includes('ongoing-request-view')">
                                              <Link class="dropdown-item" href="/ongoing-rides">
                                                  <i class="ri-taxi-fill"></i>{{ $t("ongoing-rides") }}
                                              </Link>
                                          </li>
                                          
                                          <!-- Others -->
                                          <li class="dropdown-submenu" v-if="permissions.includes('others')">
                                              <a class="dropdown-item" href="#" @click.prevent>
                                                  <i class="mdi mdi-dots-horizontal-circle"></i>{{ $t("others") }}
                                              </a>
                                              <ul class="dropdown-menu">
                                                  <li v-if="permissions.includes('view-sos')">
                                                      <Link class="dropdown-item" href="/sos">{{ $t("sos") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('view-cancellation')">
                                                      <Link class="dropdown-item" href="/cancellation">{{ $t("cancellation") }}</Link>
                                                  </li>
                                                  <li v-if="permissions.includes('view-faq')">
                                                      <Link class="dropdown-item" href="/faq">{{ $t("faq") }}</Link>
                                                  </li>
                                              </ul>
                                          </li>
                                      </ul>
                                  </li>

                                  <!-- Users Menu -->
                                  <li class="nav-item dropdown" v-if="permissions.includes('access-user-nav-list')">
                                      <a class="nav-link" href="#" @click.prevent
                                          :class="{ 'active': ['/manage-owners','/fleet-drivers/pending','/report/driver-duty-report',
                                          '/users', '/pending-drivers', '/approved-drivers','/subscription','/drivers-rating',
                                          '/admins','/referral-settings','/support-tickets'].some(path => $page.url.startsWith(path)) }">
                                          <i class="ri-user-3-line"></i>
                                          <span class="ms-2">{{ $t("users") }}</span>
                                      </a>
                                      <ul class="dropdown-menu-mega">
                                          <!-- Customer Management -->
                                          <li class="dropdown-submenu" v-if="permissions.includes('user-management')">
                                              <a class="dropdown-item" href="#" @click.prevent>
                                                  <i class="ri-team-fill"></i>{{ $t("customer-management") }}
                                              </a>
                                              <ul class="dropdown-menu">
                                                  <li v-if="permissions.includes('view-users')">
                                                      <Link class="dropdown-item" href="/users">{{ $t("user-list") }}</Link>
                                                  </li> 
                                                  <li v-if="permissions.includes('view-delete-user')">
                                                      <Link class="dropdown-item" href="/users/deleted-user">{{ $t("delete-request-users") }}</Link>
                                                  </li> 
                                                  <li v-if="permissions.includes('bulk_upload')">
                                                      <Link class="dropdown-item" href="/user-import">{{ $t("user_bulk_upload") }}</Link>
                                                  </li>
                                              </ul>
                                          </li>
                                          
                                          <!-- Driver Management -->
<li class="dropdown-submenu" v-if="permissions.includes('drivers-management')">
    <a class="dropdown-item" href="#" @click.prevent>
        <i class="ri-user-follow-line"></i>{{ $t("driver-management") }}
    </a>
    <ul class="dropdown-menu">
        <li v-if="permissions.includes('view-approval-pending-drivers')">
            <Link class="dropdown-item" href="/pending-drivers">{{ $t("pending-drivers") }}</Link>
        </li>
        <li v-if="permissions.includes('view-approved-drivers')">
            <Link class="dropdown-item" href="/approved-drivers">{{ $t("approved-drivers") }}</Link>
        </li>
        <li v-if="permissions.includes('manage-subscription')">
            <Link class="dropdown-item" href="/subscription">{{ $t("subscription") }}</Link>
        </li>
        <li v-if="permissions.includes('driver-rating-list')">
            <Link class="dropdown-item" href="/drivers-rating">{{ $t("drivers-ratings") }}</Link>
        </li>
    </ul>
</li>

<!-- Admin Management -->
<li class="dropdown-submenu" v-if="permissions.includes('admin')">
    <a class="dropdown-item" href="#" @click.prevent>
        <i class="ri-group-line"></i>{{ $t("admin-management") }}
    </a>
    <ul class="dropdown-menu">
        <li v-if="permissions.includes('admin')">
            <Link class="dropdown-item" href="/admins">{{ $t("admins") }}</Link>
        </li>
    </ul>
</li>

<!-- Referral Settings -->
<li v-if="permissions.includes('referral-settings-view')">
    <Link class="dropdown-item" href="/referral-settings">
        <i class="bx bx-share-alt"></i>{{ $t("referral-settings") }}
    </Link>
</li>

<!-- Support Management -->
<li class="dropdown-submenu" v-if="supportTicket == 1 && permissions.includes('view-support-management')">
    <a class="dropdown-item" href="#" @click.prevent>
        <i class="bx bx-support"></i>{{ $t("support_management") }}
    </a>
    <ul class="dropdown-menu">
        <li v-if="permissions.includes('view-ticket-title')">
            <Link class="dropdown-item" href="/title">{{ $t("ticket_title") }}</Link>
        </li>
        <li v-if="permissions.includes('view-support-ticket')">
            <Link class="dropdown-item" href="/support-tickets">{{ $t("support_tickets") }}</Link>
        </li>
    </ul>
</li>

<!-- Reports -->
<li class="dropdown-submenu" v-if="permissions.includes('report-management')">
    <a class="dropdown-item" href="#" @click.prevent>
        <i class="ri-file-3-fill"></i>{{ $t("report") }}
    </a>
    <ul class="dropdown-menu">
        <li v-if="permissions.includes('user-report')">
            <Link class="dropdown-item" href="/report/user-report">{{ $t("user-report") }}</Link>
        </li>
        <li v-if="permissions.includes('driver-report')">
            <Link class="dropdown-item" href="/report/driver-report">{{ $t("driver-report") }}</Link>
        </li>
    </ul>
</li>
</ul>
</li>

<!-- Settings Menu -->
<li class="nav-item dropdown" v-if="permissions.includes('access-settings-nav-list')">
    <a class="nav-link" href="#" @click.prevent
        :class="{ 'active': ['/general-settings','/wallet-settings','/payment-gateway',
        '/firebase','/map-setting','/landing-header'].some(path => $page.url.startsWith(path)) }">
        <i class="ri-settings-5-fill"></i>
        <span class="ms-2">{{ $t("settings") }}</span>
    </a>
    <ul class="dropdown-menu-mega">
        <!-- Business Settings -->
        <li class="dropdown-submenu" v-if="permissions.includes('manage-business-settings')">
            <a class="dropdown-item" href="#" @click.prevent>
                <i class="ri-settings-5-fill"></i>{{ $t("business-settings") }}
            </a>
            <ul class="dropdown-menu">
                <li v-if="permissions.includes('general-settings-view')">
                    <Link class="dropdown-item" href="/general-settings">{{ $t("general-settings") }}</Link>
                </li> 
                <li v-if="permissions.includes('customization-settings-view')">
                    <Link class="dropdown-item" href="/customization-settings">{{ $t("customization-settings") }}</Link>
                </li> 
            </ul>
        </li>
        
        <!-- App Settings -->
        <li class="dropdown-submenu" v-if="permissions.includes('manage-app-settings')">
            <a class="dropdown-item" href="#" @click.prevent>
                <i class="mdi mdi-cellphone-cog"></i>{{ $t("app-settings") }}
            </a>
            <ul class="dropdown-menu">
                <li v-if="permissions.includes('wallet-settings-view')">
                    <Link class="dropdown-item" href="/wallet-settings">{{ $t("wallet-settings") }}</Link>
                </li>
                <li v-if="permissions.includes('tip-settings-view')">
                    <Link class="dropdown-item" href="/tip-settings">{{ $t("tip-settings") }}</Link>
                </li>
            </ul>
        </li>
        
        <!-- Third Party Settings -->
        <li class="dropdown-submenu" v-if="permissions.includes('manage-third-party-settings')">
            <a class="dropdown-item" href="#" @click.prevent>
                <i class="mdi mdi-cogs"></i>{{ $t("third-party-settings") }}
            </a>
            <ul class="dropdown-menu">
                <li v-if="permissions.includes('payment-gateway-settings-view')">
                    <Link class="dropdown-item" href="/payment-gateway">{{ $t("payment-gateway-settings") }}</Link>
                </li>
                <li v-if="permissions.includes('firebase-settings-view')">
                    <Link class="dropdown-item" href="/firebase">{{ $t("firebase-settings") }}</Link>
                </li>
            </ul>
        </li>
    </ul>
</li>
</ul>
</div>   
</div>
</nav> 
</div>

<!-- Right side icons and profile with Search Bar -->
<div class="d-flex align-items-center justify-content-between px-3">
    <!-- Search Bar -->
    <div class="search-container me-3" ref="searchInputRef">
        <div class="search-input-wrapper">
            <i class="ri-search-line search-icon"></i>
            <input 
                type="text" 
                class="form-control search-input" 
                placeholder="Search menus..."
                v-model="searchQuery"
                @focus="showSearchResults = true"
                @input="showSearchResults = true"
            >
        </div>
        
        <!-- Search Results Dropdown -->
        <div v-if="showSearchResults && searchQuery" class="search-results">
            <div v-if="filteredMenuItems.length > 0" class="results-list">
                <div 
                    v-for="item in filteredMenuItems" 
                    :key="item.route"
                    class="search-result-item"
                    @click="navigateToPage(item.route)"
                >
                    <i :class="item.icon" class="result-icon"></i>
                    <div class="result-content">
                        <div class="result-name">{{ item.name }}</div>
                        <div v-if="item.parent" class="result-parent">{{ item.parent }}</div>
                    </div>
                    <i class="ri-arrow-right-s-line result-arrow"></i>
                </div>
            </div>
            <div v-else class="no-results">
                <i class="ri-search-line"></i>
                <p>No results found</p>
            </div>
        </div>
    </div>

    <!-- Profile Dropdown -->
    <BDropdown variant="link" class="ms-sm-3 header-item topbar-user"
        toggle-class="rounded-circle arrow-none" menu-class="dropdown-menu-end"
        :offset="{ alignmentAxis: -14, crossAxis: 0, mainAxis: 0 }">
        <template #button-content>
            <span class="d-flex align-items-center">
                <img v-if="$page.props.jetstream.managesProfilePhotos"
                    class="rounded-circle header-profile-user"
                    :src="profilePhotoUrl || '/default-profile.jpeg'" :alt="$page.props.auth.user.name">
                <span class="text-start ms-xl-2">
                    <span class="d-none d-xl-inline-block ms-1 fw-medium user-name-text">{{
                        $page.props.auth.user.name }}</span>
                </span>
            </span>
        </template>
        <h6 class="dropdown-header">{{$t("welcome")}} {{ $page.props.auth.user.name }}!</h6>
        <Link class="dropdown-item" href="/profile-edit"><i
            class="mdi mdi-account-circle text-muted fs-16 align-middle me-1"></i>
        <span class="align-middle">{{$t("profile")}}</span>
        </Link>
        <form method="POST" @submit.prevent="logout" class="dropdown-item">
            <BButton variant="none" type="submit" class="btn p-0"><i
                    class="mdi mdi-logout text-muted fs-16 align-middle me-1"></i> {{$t("logout")}}</BButton>
        </form>
    </BDropdown>
</div>
</div>
</div>
</header>
</template>

<style scoped>
/* Search Container Styles */
.search-container {
  position: relative;
  width: 280px;
}

.search-input-wrapper {
  position: relative;
}

.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #878a99;
  font-size: 18px;
  pointer-events: none;
}

.search-input {
  padding-left: 40px;
  height: 38px;
  border-radius: 4px;
  border: 1px solid #e9ecef;
  font-size: 14px;
  transition: all 0.3s ease;
}

.search-input:focus {
  border-color: #405189;
  box-shadow: 0 0 0 0.2rem rgba(64, 81, 137, 0.1);
  outline: none;
}

/* Search Results Dropdown */
.search-results {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  right: 0;
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 6px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  max-height: 400px;
  overflow-y: auto;
  z-index: 9999;
}

.results-list {
  padding: 8px 0;
}

.search-result-item {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  border-bottom: 1px solid #f3f6f9;
}

.search-result-item:last-child {
  border-bottom: none;
}

.search-result-item:hover {
  background-color: #f8f9fa;
}

.result-icon {
  font-size: 18px;
  color: #405189;
  width: 24px;
  flex-shrink: 0;
}

.result-content {
  flex: 1;
  margin-left: 12px;
}

.result-name {
  font-size: 14px;
  font-weight: 500;
  color: #495057;
}

.result-parent {
  font-size: 12px;
  color: #878a99;
  margin-top: 2px;
}

.result-arrow {
  font-size: 18px;
  color: #878a99;
  flex-shrink: 0;
}

.no-results {
  text-align: center;
  padding: 32px 16px;
  color: #878a99;
}

.no-results i {
  font-size: 48px;
  margin-bottom: 12px;
  opacity: 0.5;
}

.no-results p {
  margin: 0;
  font-size: 14px;
}

/* Rest of your existing styles */
.btn-check:checked + .btn, 
:not(.btn-check) + .btn:active, 
.btn:first-child:active, 
.btn.active, 
.btn.show {
    background-color: var(--btn-active-border-color) !important;
}

/* ... rest of existing styles ... */

@media screen and (max-width: 769px) {
    .search-container {
        width: 180px;
    }
}

@media screen and (max-width: 576px) {
    .search-container {
        width: 150px;
    }
}
</style>
<style scoped>
      /* Base button and active states */
      .btn-check:checked + .btn, 
        :not(.btn-check) + .btn:active, 
        .btn:first-child:active, 
        .btn.active, 
        .btn.show {
            background-color: var(--btn-active-border-color) !important;
        }

        .service-location {
            display: inline-block;
            width: 59px;
            white-space: nowrap;
            overflow: hidden !important;
            text-overflow: ellipsis;
        }

        /* Active navigation state */
        .top-navbar .nav-item .active {
            font-weight: 600;
            color: var(--side_menu) !important;
            font-size: 16px;
        }

        /* Level 1: Main navigation */
        .navbar-nav {
            display: flex;
            list-style: none;
            margin: 0;
            padding: 0;
            align-items: center;
        }

        .nav-item {
            margin: 0 0.5rem;
            position: relative;
        }

        .nav-link {
            color: #e6e2e2f1;
            padding: 0.5rem 1rem;
            text-decoration: none;
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }

        /* Level 2: Mega menu container - HORIZONTAL */
        .dropdown-menu-mega {
            position: absolute;
            top: 100%;
            left: 0;
            min-width: 800px;
            max-width: 1200px;
            padding: 1rem;
            background: #fff;
            border: 1px solid rgba(0,0,0,.15);
            border-radius: 0.375rem;
            box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.175);
            display: none;
            flex-direction: row;
            flex-wrap: wrap;
            gap: 0.5rem;
            align-items: flex-start;
            z-index: 1000;
        }

        /* Show dropdown on hover */
        .nav-item.dropdown:hover > .dropdown-menu-mega {
            display: flex !important;
        }

        /* Level 2: Direct children horizontal */
        .dropdown-menu-mega > li {
            flex: 0 0 auto;
            position: relative;
            list-style: none;
        }

        /* Level 2: Menu items - horizontal pills */
        .dropdown-menu-mega > li > .dropdown-item {
            font-weight: 500;
            color: #495057;
            padding: 0.6rem 1rem;
            cursor: pointer;
            background-color: #fff;
            border: 1px solid #e9ecef;
            border-radius: 0.25rem;
            white-space: nowrap;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            transition: all 0.2s ease;
            text-decoration: none;
        }

        .dropdown-menu-mega > li > .dropdown-item:hover {
            background-color: #f8f9fa;
            color: #405189;
            border-color: #405189;
        }

        /* Arrow for items with submenu */
        .dropdown-submenu > .dropdown-item::after {
            content: "\25BC";
            font-size: 0.7rem;
            margin-left: 0.25rem;
            transition: transform 0.2s ease;
        }

        .dropdown-submenu:hover > .dropdown-item::after {
            transform: rotate(180deg);
        }

        /* Level 3: Sub-submenu container */
        .dropdown-submenu > .dropdown-menu {
            position: absolute;
            left: 0;
            top: 100%;
            margin-top: 0.5rem;
            padding: 0.75rem;
            border: 1px solid rgba(0,0,0,.15);
            border-radius: 0.375rem;
            box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.175);
            background: #fff;
            display: none;
            min-width: 300px;
            z-index: 1001;
            flex-direction: row;
            flex-wrap: wrap;
            gap: 0.5rem;
            list-style: none;
            padding-left: 0;
        }

        /* Show level 3 on hover */
        .dropdown-submenu:hover > .dropdown-menu {
            display: flex !important;
        }

        /* Level 3: Child items - horizontal pills */
        .dropdown-submenu .dropdown-menu .dropdown-item {
            padding: 0.5rem 1rem;
            font-size: 0.875rem;
            color: #6c757d;
            font-weight: 400;
            background-color: #fff;
            border: 1px solid #e9ecef;
            border-radius: 0.25rem;
            white-space: nowrap;
            flex: 0 0 auto;
            transition: all 0.2s ease;
            text-decoration: none;
        }

        .dropdown-submenu .dropdown-menu .dropdown-item:hover {
            background-color: #405189;
            color: #fff;
            border-color: #405189;
            transform: translateY(-2px);
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        /* Icon styling */
        .dropdown-item i {
            width: 18px;
            text-align: center;
        }

        .dropdown-menu-mega > li > .dropdown-item i {
            color: #405189;
        }

        .text-truncate {
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }

        /* Responsive */
        @media screen and (max-width: 769px) {
            .dropdown-menu-mega {
                min-width: 280px;
                flex-direction: column;
            }
            
            .dropdown-menu-mega > li {
                width: 100%;
            }
            
            .dropdown-submenu .dropdown-menu {
                position: static;
                margin-top: 0.5rem;
                margin-left: 1rem;
            }
        }
</style>