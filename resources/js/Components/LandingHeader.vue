<script>
import { CountTo } from "vue3-count-to";
import { Link } from '@inertiajs/vue3';
import { Autoplay, Navigation, Pagination } from "swiper/modules";
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/css";
import "swiper/css/autoplay";
import 'swiper/css/navigation';
import 'swiper/css/pagination';

export default {
    
    data() {
        return {
            Autoplay, Navigation, Pagination,
            currentTab: '',
            isCollapsed: false, // Track the collapse state
            header: window.headers, // Access global headers data
            headers: this.$page.props.landingHeader,
            enable_web_booking: window.headers[0].enable_web_booking,
            locales: this.$page.props.locales,
            selectedLocale: this.$page.props.landingHeader.locale,
            selectedDirection: this.$page.props.landingHeader.direction,
            user_login: window.headers[0].userlogin,
        };
    },
    components: {
        Swiper,
        SwiperSlide,
        CountTo,
        Link,
    },
    methods: {
        toggleMenu() {
            this.isCollapsed = !this.isCollapsed; // Toggle collapse state
        },
        changeLocale(event) {
            const localeId = event.target.value;
            this.selectedLocale = this.locales[localeId];
            // localStorage.setItem('locale', true)
            window.location.href = `?locale=${this.selectedLocale}`;
        },

        changeLocale(locale) {
        this.selectedLocale = locale;
        
        window.location.href = `?locale=${this.selectedLocale}`;
        },

        headerLogoUrl() {
            const selectedHeader = this.header.find(header => header.locale === this.selectedLocale);
            return selectedHeader ? selectedHeader.header_logo_url : '';
        },

        // headerLogoUrl() {
        //     return this.header.length > 0 ? this.header[1].header_logo_url : '';
        // },

    },

    mounted(){
        const body = document.body;
        if( this.selectedDirection === 'rtl'){
            localStorage.setItem('directiontoggleValue', true);
            body.classList.add('rtl');
             body.classList.remove('ltr');
        }
        else{
            localStorage.setItem('directiontoggleValue', false);
            body.classList.add('ltr');
            body.classList.remove('rtl');
        }
    },
    created() {
        // Set initial active tab based on current route
        this.currentTab = window.location.pathname;
    },
};
</script>

<template>
    <nav class="navbar navbar-expand-lg navbar-landing fixed-top" style="background-color: var(--landing_header_bg);" id="navbar">
        <BContainer>
            <Link class="navbar-brand">
                <img :src="headerLogoUrl()" class="card-logo card-logo-dark" alt="logo light" width="70">
            </Link>
            <BButton variant="link" class="navbar-toggler py-0 fs-20 text-body" @click="toggleMenu()">
                <i class="mdi mdi-menu"></i>
            </BButton>

            <BCollapse class="navbar-collapse" id="navbarSupportedContent" v-model="isCollapsed">
                <ul class="navbar-nav mx-auto mt-2 mt-lg-0" id="navbar-example">
                    <li class="nav-item">
                        <Link class="nav-link" :class="{ 'active': currentTab === '/' }" href="/">{{ headers.home }}</Link>
                    </li>
                    <li class="nav-item">
                        <Link class="nav-link" :class="{ 'active': currentTab === '/aboutus' }" href="/aboutus">{{ headers.aboutus }}</Link>
                    </li>
                    <li class="nav-item">
                        <Link class="nav-link" :class="{ 'active': currentTab === '/driver' }" href="/driver">{{ headers.driver }}</Link>
                    </li>
                    <li class="nav-item">
                        <Link class="nav-link" :class="{ 'active': currentTab === '/user' }" href="/user">Usersssss</Link>
                    </li>
                    <li class="nav-item">
                        <Link class="nav-link" :class="{ 'active': currentTab === '/contact' }" href="/contact">{{ headers.contact }}</Link>
                    </li>
                </ul>
                <div class="flex-shrink-0 me-5">
              
                   
                </div>
                <div class="">
                    <BLink v-if="enable_web_booking == 1" class="btn text-white" style="background-color: var(--landing_header_act_text);" :href="user_login">{{ headers.book_now_btn }}</BLink>
                </div>
            </BCollapse>
        </BContainer>
    </nav>
</template>

