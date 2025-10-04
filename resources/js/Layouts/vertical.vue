<script>
import { layoutComputed } from "@/state/helpers";

import NavBar from "@/Components/nav-bar.vue";
import RightBar from "@/Components/right-bar.vue";
import Footer from "@/Components/footer.vue";
localStorage.setItem('hoverd', false);

/**
 * Vertical layout
 */
export default {
  components: { NavBar, RightBar, Footer },
  // props: {
  //   activeMenu: {
  //     type: String,
  //     required: true
  //   },
  // },
  data() {
    return {
      isMenuCondensed: false,
      activeMenu:'',
      supportTicket:window.supportTicket
    };
  },
  computed: {
    ...layoutComputed,
    sidebarSize: {
      get() {
        return this.$store ? this.$store.state.layout.sidebarSize : {} || {};
      },
      set(type) {
        return this.changeSidebarSize({
          sidebarSize: type,
        });
      },
    },
  },
  created: () => {
    document.body.removeAttribute("data-layout", "horizontal");
    document.body.removeAttribute("data-topbar", "dark");
    document.body.removeAttribute("data-layout-size", "boxed");
  },
  methods: {
    toggleRightSidebar() {
      document.body.classList.toggle("right-bar-enabled");
    },
    hideRightSidebar() {
      document.body.classList.remove("right-bar-enabled");
    },   
  },
  mounted() {
    // Set active menu based on current path for navbar highlighting
    const path = window.location.pathname;   

    if (path.startsWith('/general-settings') || path.startsWith('/wallet-settings') || path.startsWith('/tip-settings') 
    || path.startsWith ('/transport-ride-settings') || path.startsWith ('/bid-ride-settings')
    || path.startsWith('/payment-gateway') || path.startsWith('/sms-gateway') || path.startsWith('/app_modules')
    || path.startsWith('/firebase') || path.startsWith('/map-setting') || path.startsWith('/peakzone-setting')
    || path.startsWith('/landing-home')  || path.startsWith('/landing-driver') ||path.startsWith('/country') ||
    path.startsWith('/landing-user') || path.startsWith('/landing-contact') || path.startsWith('/landing-header') || path.startsWith('/onboarding-screen')
    || path.startsWith('/landing-home') || path.startsWith('/landing-header') || path.startsWith('/landing-quicklink') || path.startsWith('/onboarding-screen')
    || path.startsWith('/invoice-configuration') || path.startsWith('/mail-configuration') || path.startsWith('/map-apis') || path.startsWith('/recaptcha')
    || path.startsWith('/landing-aboutus') || path.startsWith('/notification-channel') || path.startsWith('/customization-settings'))
    {
      this.activeMenu = 'Settings';
    } 
    else if ((path.startsWith('/languages')) 
    || (path.startsWith('/roles'))  || (path.startsWith('/permissions')) || (path.startsWith('/preferences')))
    {
      this.activeMenu = 'Masters';
    } 
    else if (path.startsWith('/manage-owners') 
     || path.startsWith('/owner-needed-documents') || path.startsWith('/report/owner-report') 
     || path.startsWith('/report/finance-report') || path.startsWith('/roles') || path.startsWith('/admins') 
     || path.startsWith('/users') || path.startsWith('/users/deleted-user') || path.startsWith('/report/user-report')
     || path.startsWith('/negative-balance-drivers') || path.startsWith('/withdrawal-request-drivers') || path.startsWith('/delete-request-drivers')
     || path.startsWith('/driver-needed-documents') || path.startsWith('/report/driver-report') || path.startsWith('/user-dashboard') || path.startsWith('/drivers-rating')
     || path.startsWith('/fleet-drivers')  || path.startsWith('/fleet-drivers/pending')  || path.startsWith('/fleet-needed-documents')
     || path.startsWith('/manage-fleet') || path.startsWith('/pending-drivers') || path.startsWith('/approved-drivers')  || path.startsWith('/fleet-drivers') 
     || path.startsWith('/driver-bank-info') || path.startsWith('/report/fleet-report')
     || path.startsWith('/report/driver-duty-report') || path.startsWith('/subscription')
     || path.startsWith('/referral-settings') || path.startsWith('/owner-dashboard') || path.startsWith('/withdrawal-request-owners') ||
        path.startsWith('/category') ||  path.startsWith('/title')||  path.startsWith('/support-tickets') ||(path.startsWith('/user-import')) || (path.startsWith('/driver-import'))
    ) 
    {
      this.activeMenu = 'Users';
    }
    else{
      this.activeMenu = 'Home';
    }
  }
};
</script>
  
<template>
  <div id="layout-wrapper">
    <NavBar />
    <!-- ============================================================== -->
    <!-- Start Page Content here -->
    <!-- ============================================================== -->
    <div class="main-content">
      <div class="page-content">
        <!-- Start Content-->
        <b-container fluid>
          <slot />
        </b-container>
      </div>
      <Footer />
    </div>
    <RightBar />
  </div>
</template>