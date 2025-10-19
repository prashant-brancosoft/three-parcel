<script>
import { CountTo } from "vue3-count-to";
import { Autoplay, Navigation, Pagination } from "swiper/modules";
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/css";
import "swiper/css/autoplay";
import 'swiper/css/navigation';
import 'swiper/css/pagination';
import LandingHeader from "@/Components/LandingHeader.vue";
import LandingFooter from "@/Components/LandingFooter.vue";
import { Link, router,Head } from '@inertiajs/vue3';



export default {
    
    props: {
        landingHome: Object,
        landingHeader: Object,
    },
    computed: {
    splitServices() {
      // Check if landingHome.data is a string and split it by commas
      if (typeof this.landingHome.services === 'string') {
        return this.landingHome.services.split(',');
      }
      // Return an empty array if data is not a string
      return [];
    },
    splitAbout() {
      // Check if landingHome.data is a string and split it by commas
      if (typeof this.landingHome.about_lists === 'string') {
        return this.landingHome.about_lists.split(',');
      }
      // Return an empty array if data is not a string
      return [];
    },
  },
    data() {
        return {
            Autoplay, Navigation, Pagination,
        };
    },
    methods: {
        stripHtmlTags(content) {
            const parser = new DOMParser();
            const parsedContent = parser.parseFromString(content, 'text/html');
            return parsedContent.body.textContent || "";
            },
    },

    components: {
        Swiper,
        SwiperSlide,
        CountTo,
        LandingHeader,
        LandingFooter,
        Link,
        router,
        Head
    },
};
</script>

<template>
    <section class="creative-template">
		<div class="coming-soon-content">
			<h6>COMING SOON</h6>
			<h1>We’r blowing up</h1>
			<div class="row">
				<div class="col-md-9">
					<p>We're under construction. Check back for an update soon. Stay in touch!</p>
				</div>
			</div>
			<form action="#">
				<input type="email" placeholder="threeparcel@gmail.com">
				<button>Notify Me</button>
			</form>

			<ul class="social-icon">
				<li><a href="https://www.fiverr.com/sellers/creativesalahu/"><i class="fab fa-facebook-f"></i></a></li>
				<li><a href="https://www.fiverr.com/sellers/creativesalahu/"><i class="fab fa-instagram"></i></a></li>
				<li><a href="https://www.fiverr.com/sellers/creativesalahu/"><i class="fab fa-pinterest-p"></i></a></li>
				<li><a href="https://www.fiverr.com/sellers/creativesalahu/"><i class="fab fa-linkedin-in"></i></a></li>
			</ul>
		</div>
		<img src="https://i.ibb.co/Xfkk7380/shape-46-creativesalahu.png" alt="" class="shapes shape-one">
		<img src="https://i.ibb.co/W4kRJs0T/shape-47-creativesalahu.png" alt="" class="shapes shape-two">
		<img src="https://i.ibb.co/yFPBGfWd/shape-48-creativesalahu.png" alt="" class="shapes shape-three">
		<img src="https://i.ibb.co/rft2vzg0/shape-49-creativesalahu.png" alt="" class="shapes shape-four">
		<img src="https://i.ibb.co/35q6g9yS/shape-50-creativesalahu.png" alt="" class="shapes shape-five">
		<img src="https://i.ibb.co/35q6g9yS/shape-50-creativesalahu.png" alt="" class="shapes shape-six">
	</section>
<!-- Buy Me A Coffee -->
<a href="#" class="coffee-button" target="_blank">
    <span class="coffee-icon">☕</span> Parcel Delivery
  </a>

    <div class="layout-wrapper landing d-none">
        <Head :title="$t('home')" />
        <LandingHeader :headers="landingHeader">
            
        </LandingHeader>
        
        <div class="vertical-overlay" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent.show"></div>
        <section class="section pb-0 hero-section" id="hero">
            <div class="bg-overlay bg-overlay-pattern"></div>
            <BContainer class="download-app">
                <BRow class="justify-content-center">
                    <BCol lg="8" sm="10">
                        <div class="text-center mt-lg-5 pt-5">
                            <h1 class="display-6 fw-semibold mb-3 lh-base">{{ landingHome.hero_title }}</h1>
                            <div class="d-flex align-items-center justify-content-center gap-4 mb-5">
                            <div class="d-block">
                            <h2  style="color: var(--landing_header_act_text);">{{ landingHeader.user_app }}</h2>

                            <div class="d-flex gap-2 justify-content-center mt-4">
                                <BLink :href="landingHome.hero_user_link_android" target="_blank" class="btn btn-primary">{{ landingHeader.user_play }} <i
                                        class="ri-google-play-fill align-middle ms-1"></i></BLink>
                                <BLink :href="landingHome.hero_user_link_apple" target="_blank" class="btn btn-dark">{{ landingHeader.user_apple }} <i
                                        class="ri-apple-fill align-middle ms-1"></i></BLink>
                            </div>
                            </div>
                        <div class="d-block">
                            <h2 style="color: var(--landing_header_act_text);">{{ landingHeader.driver_app }}</h2>

                            <div class="d-flex gap-2 justify-content-center mt-4">
                                <BLink :href="landingHome.hero_driver_link_android" target="_blank" class="btn btn-primary">{{ landingHeader.driver_play }} <i
                                        class="ri-google-play-fill align-middle ms-1"></i></BLink>
                                <BLink :href="landingHome.hero_driver_link_apple" target="_blank" class="btn btn-dark">{{ landingHeader.driver_apple }} <i
                                        class="ri-apple-fill align-middle ms-1"></i></BLink>
                            </div>
                        </div>
                            </div>
                        </div>
                    </BCol>
                </BRow>
            </BContainer>
        </section>

       

        <section class="section" id="services">
            <BContainer>
                <BRow class="justify-content-center">
                    <BCol lg="8">
                        <div class="text-center mb-5">
                            <h1 class="mb-3 ff-secondary fw-semibold lh-base">{{ landingHome.feature_heading }}</h1>
                            <p class="text-muted">{{ stripHtmlTags(landingHome.feature_para) }}</p>
                        </div>
                    </BCol>
                </BRow>

                <BRow class="g-3 advantages">
                    <BCol lg="6">
                        <div class="d-flex p-3">
                            <div class="flex-shrink-0 me-3">
                                <div class="avatar-sm icon-effect">
                                    <div class="avatar-title bg-transparent  rounded-circle" style="color: var(--landing_header_act_text);">
                                        <i class="ri-pencil-ruler-2-line fs-36"></i>
                                    </div>
                                </div>
                            </div>
                            <div class="flex-grow-2">
                                <h5 class="fs-18">{{ landingHome.feature_sub_heading_1 }}</h5>
                                <p class="text-muted my-3 ff-secondary">{{ stripHtmlTags(landingHome.feature_sub_para_1) }}</p>
                               
                            </div>
                        </div>
                    </BCol>
                    <BCol lg="6">
                        <div class="d-flex p-3">
                            <div class="flex-shrink-0 me-3">
                                <div class="avatar-sm icon-effect">
                                    <div class="avatar-title bg-transparent  rounded-circle" style="color: var(--landing_header_act_text);">
                                        <i class="ri-palette-line fs-36"></i>
                                    </div>
                                </div>
                            </div>
                            <div class="flex-grow-1">
                                <h5 class="fs-18">{{ landingHome.feature_sub_heading_2 }}</h5>
                                <p class="text-muted my-3 ff-secondary">{{ stripHtmlTags(landingHome.feature_sub_para_2) }}</p>
                                
                            </div>
                        </div>
                    </BCol>
                    <BCol lg="6">
                        <div class="d-flex p-3">
                            <div class="flex-shrink-0 me-3">
                                <div class="avatar-sm icon-effect">
                                    <div class="avatar-title bg-transparent  rounded-circle" style="color: var(--landing_header_act_text);">
                                        <i class="ri-lightbulb-flash-line fs-36"></i>
                                    </div>
                                </div>
                            </div>
                            <div class="flex-grow-1">
                                <h5 class="fs-18">{{ landingHome.feature_sub_heading_3 }}</h5>
                                <p class="text-muted my-3 ff-secondary">{{ stripHtmlTags(landingHome.feature_sub_para_3) }}</p>
                                
                            </div>
                        </div>
                    </BCol>
                    <BCol lg="6">
                        <div class="d-flex p-3">
                            <div class="flex-shrink-0 me-3">
                                <div class="avatar-sm icon-effect">
                                    <div class="avatar-title bg-transparent  rounded-circle" style="color: var(--landing_header_act_text);">
                                        <i class="ri-customer-service-line fs-36"></i>
                                    </div>
                                </div>
                            </div>
                            <div class="flex-grow-1">
                                <h5 class="fs-18">{{ landingHome.feature_sub_heading_4 }}</h5>
                                <p class="text-muted my-3 ff-secondary">{{ stripHtmlTags(landingHome.feature_sub_para_4) }}</p>
                               
                            </div>
                        </div>
                    </BCol>
                </BRow>
            </BContainer>
        </section>



        <section class="section">
            <BContainer>
                <BRow class="align-items-center gy-4">
                    <BCol lg="6" order="2" order-lg="1" class="digitalservice">
                        <div class="text-muted">
                            <h5 class="fs-12 text-uppercase" style="color: var(--landing_header_act_text);">{{ landingHome.service_heading_1 }}</h5>
                            <h4 class="mb-3">{{ landingHome.service_heading_2 }}</h4>
                            <p class="mb-4 ff-secondary">{{ stripHtmlTags(landingHome.service_para) }}</p>

                            <BRow>
                                <BCol sm="12">
                                    <div class="vstack gap-2">
                                        <div class="d-flex align-items-center" v-for="(item, index) in splitServices" :key="index">
                                            <div class="flex-shrink-0 me-2">
                                                <div class="avatar-xs icon-effect">
                                                    <div class="avatar-title bg-transparent rounded-circle h2" style="color: var(--landing_header_act_text);">
                                                        <i class="ri-check-fill"></i>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="flex-grow-1">
                                                <h5 class="fs-14 mb-0">{{ stripHtmlTags(item) }}</h5>
                                            </div>
                                        </div>
                                    </div>
                                </BCol>
                            </BRow>
                        </div>
                    </BCol>
                    <BCol lg="6" sm="7" cols="10" order="1" order-lg="2" class="ms-auto">
                        <div>
                            <img :src="landingHome.service_img_url" alt="" class="img-fluid">
                        </div>
                    </BCol>
                </BRow>

                <BRow class="align-items-center mt-5 pt-lg-5 gy-4">
                    <BCol lg="6" sm="7" cols="10" class="mx-auto">
                        <div>
                            <img :src="landingHome.about_img_url" alt="" class="img-fluid">
                        </div>
                    </BCol>
                    <BCol lg="6" class="aboutcompany">
                        <div class="text-muted ps-lg-5">
                            <h5 class="fs-12 text-uppercase" style="color: var(--landing_header_act_text);">{{ landingHome.about_title_1 }}</h5>
                            <h4 class="mb-3">{{ landingHome.about_title_2 }}</h4>
                            <p class="mb-4">{{ stripHtmlTags(landingHome.about_para) }}</p>

                            <div class="vstack gap-2">
                                <div class="d-flex align-items-center" v-for="(item, index) in splitAbout" :key="index">
                                    <div class="flex-shrink-0 me-2">
                                        <div class="avatar-xs icon-effect">
                                            <div class="avatar-title bg-transparent  rounded-circle h2" style="color: var(--landing_header_act_text);">
                                                <i class="ri-check-fill"></i>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="flex-grow-1">
                                        <p class="mb-0">{{ stripHtmlTags(item) }}</p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </BCol>
                </BRow>
            </BContainer>
        </section>

        <section class="section bg-light" id="team">
            <BContainer>
                <BRow>
                    <BCol lg="4" sm="6">
                        <BCard n-body>
                            <BCardBody class="text-center p-4">
                                <div class="avatar-xxl mx-auto mb-4 position-relative">
                                    <img :src="landingHome.box_img_1_url" alt="" class="img-fluid rounded-circle">
                                </div>
                                <h5 class="mb-1">{{ stripHtmlTags(landingHome.box_para_1) }}</h5>
                            </BCardBody>
                        </BCard>
                    </BCol>
                    <BCol lg="4" sm="6">
                        <BCard n-body>
                            <BCardBody class="text-center p-4">
                                <div class="avatar-xxl mx-auto mb-4 position-relative">
                                    <img :src="landingHome.box_img_2_url" alt="" class="img-fluid rounded-circle">
                                </div>
                                <h5 class="mb-1">{{ stripHtmlTags(landingHome.box_para_2) }}</h5>
                            </BCardBody>
                        </BCard>
                    </BCol>
                    <BCol lg="4" sm="6">
                        <BCard n-body>
                            <BCardBody class="text-center p-4">
                                <div class="avatar-xxl mx-auto mb-4 position-relative">
                                    <img :src="landingHome.box_img_3_url" alt="" class="img-fluid rounded-circle">
                                </div>
                                <h5 class="mb-1">{{ stripHtmlTags(landingHome.box_para_3) }}</h5>
                            </BCardBody>
                        </BCard>
                    </BCol>
                </BRow>
            </BContainer>
        </section>


        <section class="section">
            <BContainer>
                <BRow class="justify-content-center">
                    <BCol lg="8">
                        <div class="text-center mb-5">
                            <h3 class="mb-3 fw-semibold">{{ landingHome.drive_heading }}</h3>
                        </div>
                    </BCol>
                </BRow>

                <BRow class="text-center whyusTagxi">
                    <BCol lg="4">
                        <div class="process-card mt-4">
                            <div class="process-arrow-img d-none d-lg-block">
                                <!-- <img src="@/assets/images/landing/process-arrow-img.png" alt="" class="img-fluid"> -->
                            </div>
                            <div class="avatar-sm icon-effect mx-auto mb-4">
                                <div class="avatar-title bg-transparent rounded-circle h1" style="color: var(--landing_header_act_text);">
                                    <i class="ri-quill-pen-line"></i>
                                </div>
                            </div>

                            <h5>{{ landingHome.drive_title_1 }}</h5>
                            <p class="text-muted ff-secondary">{{ stripHtmlTags(landingHome.drive_para_1) }}</p>
                        </div>
                    </BCol>
                    <BCol lg="4">
                        <div class="process-card mt-4">
                            <div class="process-arrow-img d-none d-lg-block">
                                <!-- <img src="@/assets/images/landing/process-arrow-img.png" alt="" class="img-fluid"> -->
                            </div>
                            <div class="avatar-sm icon-effect mx-auto mb-4">
                                <div class="avatar-title bg-transparent  rounded-circle h1" style="color: var(--landing_header_act_text);">
                                    <i class="ri-user-follow-line"></i>
                                </div>
                            </div>

                            <h5>{{ landingHome.drive_title_2 }}</h5>
                            <p class="text-muted ff-secondary">{{ stripHtmlTags(landingHome.drive_para_2) }}</p>
                        </div>
                    </BCol>
                    <BCol lg="4">
                        <div class="process-card mt-4">
                            <div class="avatar-sm icon-effect mx-auto mb-4">
                                <div class="avatar-title bg-transparent  rounded-circle h1" style="color: var(--landing_header_act_text);">
                                    <i class="ri-book-mark-line"></i>
                                </div>
                            </div>

                            <h5>{{ landingHome.drive_title_3 }}</h5>
                            <p class="text-muted ff-secondary">{{ stripHtmlTags(landingHome.drive_para_3) }}</p>
                        </div>
                    </BCol>
                </BRow>
            </BContainer>
        </section>



        <section class="section bg-light py-5" id="features">
            <BContainer>
                <BRow class="align-items-center gy-4">
                    <BCol lg="6" sm="7" class="mx-auto">
                        <div>
                            <img :src="landingHome.service_area_img_url" alt="location" class="img-fluid mx-auto">
                        </div>
                    </BCol>
                    <BCol lg="6" class="servicelocation">
                        <div class="text-muted">
                            <div class="avatar-sm icon-effect mb-4">
                                <div class="avatar-title bg-transparent rounded-circle h1" style="color: var(--landing_header_act_text);">
                                    <i class="ri-collage-line fs-36"></i>
                                </div>
                            </div>
                            <h3 class="mb-3 fs-20">{{ landingHome.service_area_title }}</h3>
                            <p class="mb-4 ff-secondary fs-16">{{ stripHtmlTags(landingHome.service_area_para) }}</p>

                        </div>
                    </BCol>
                </BRow>
            </BContainer>
        </section>

<LandingFooter></LandingFooter>
    </div>
</template>

<style>
.icon-effect::before {
    background-color:  rgb(from var(--landing_header_act_text) r g b / 33%);
}

/* new */
@import url('https://fonts.googleapis.com/css2?family=Libre+Baskerville:ital,wght@0,400;0,700;1,400&family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap');
body{
	font-weight: normal;
	font-size: 18px;
	position: relative;
	line-height: 1.66em;
	color: #292929;
	background-color: #fff;
	-webkit-text-size-adjust: 100%;
	-webkit-tap-highlight-color: transparent;
}
* {
	margin: 0;
	padding: 0;
	-webkit-box-sizing: border-box;
	-moz-box-sizing: border-box;
	box-sizing: border-box;
}
*, ::after, ::before {
	box-sizing: border-box;
}
img {
    max-width: 100%;
    display: block;
}
img, svg {
    vertical-align: middle;
}
/* ================= Coming Soon Template ================== */
.creative-template {
    min-height: 100vh;
    padding: 40px 12px;
    position: relative;
    z-index: 5;
    background: url(https://imgpanda.com/upload/ib/0pOfzusR8z.png) no-repeat center;
    background-size: cover;
    overflow-x: hidden;
    display: flex;
    align-items: center;
    width: 100%;
}
.coming-soon-content {
    max-width: 860px;
    margin: 0 auto;
    text-align: center;
}
.coming-soon-content h6{
	    font-family: "Roboto", sans-serif;
    font-size: 16px;
    text-transform: uppercase;
    letter-spacing: 2px;
    color: #000;
    font-weight: normal;
    margin-top: 0;
    margin-bottom: .5rem;
    line-height: 1.2;
}
.coming-soon-content h1{
	    font-size: 100px;
    line-height: 1.18em;
    padding: 8px 0 30px;
    color: #000;
    font-weight: normal;
    font-family: "Libre Baskerville", serif;
    margin-top: 0;
    margin-bottom: .5rem;
}
.coming-soon-content .row {
    display: flex;
    flex-wrap: wrap;
}
.coming-soon-content .row .col-md-9 {
    flex: 0 0 auto;
    width: 75%;
    margin-left: auto;
    margin-right: auto;
}
.coming-soon-content .row .col-md-9 p{
	    font-size: 24px;
    line-height: 1.66em;
    padding-bottom: 55px;
    font-family: "Roboto", sans-serif;
    margin-top: 0;
    margin-bottom: 1rem;
    font-weight: 300;
}
.coming-soon-content form{
	    max-width: 620px;
    height: 75px;
    position: relative;
    margin: 0 auto;
}
.coming-soon-content form input{
	    display: block;
    width: 100%;
    border: none;
    background: #F4F4F4;
    border-radius: 10px;
    padding: 0 190px 0 20px;
    height: 100%;
    box-shadow: 0 25px 50px rgba(0, 0, 0, .05);
    outline: none;
    transition: all .3s ease-in-out;
    font-size: 18px;
    letter-spacing: 0.5px;
    line-height: normal;
    font-family: "Roboto", sans-serif;
}
.coming-soon-content form button{
    width: 185px;
    background: #151515;
    color: #fff;
    border-radius: 10px;
    position: absolute;
    top: 6px;
    right: 6px;
    bottom: 6px;
    transition: all .3s ease-in-out;
    font-family: "Roboto", sans-serif;
    font-size: 18px;
    border: none;
    cursor: pointer;
}
.coming-soon-content form button:hover{
	    background: #ff2759;
}
.coming-soon-content .social-icon {
    position: relative;
    list-style: none;
    padding-left: 0;
    margin-bottom: 0;
    display: flex;
    justify-content: center;
    margin-top: 40px;
}
.coming-soon-content .social-icon li a{
width: 34px;
    height: 34px;
    line-height: 32px;
    font-size: 17px;
    text-align: center;
    background: #fff;
    border-radius: 7px;
    color: #333333;
    margin: 0 5px;
    transition: all .3s ease-in-out;
    text-decoration: none;
    display: inline-block;
}
.shapes{
	    position: absolute;
    z-index: -1;
}
.creative-template .shape-one {
    top: 8%;
    left: 45%;
    animation: jumpTwo 5s infinite linear;
}
.creative-template .shape-two{
top: 6%;
    right: 45%;
    animation: jumpThree 5s infinite linear;
}
.creative-template .shape-three{
    top: 20%;
    right: 8%;	
}
.creative-template .shape-four{
    bottom: 20%;
    left: 8%;	
}
.creative-template .shape-five{
	    top: 20%;
    left: 10%;
    animation: moveclouds 10s alternate infinite;
}
.creative-template .shape-six{
	    bottom: 20%;
    right: 10%;
    animation: moveclouds2 10s alternate infinite;
}
@keyframes jumpTwo{
	0% {
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
}
40% {
    -webkit-transform: translate3d(0, 20px, 0);
    transform: translate3d(0, 20px, 0);
}
100% {
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
}
}
@keyframes jumpThree{
	0% {
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
}

40% {
    -webkit-transform: translate3d(0, -20px, 0);
    transform: translate3d(0, -20px, 0);
}
100% {
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
}
}
@keyframes moveclouds{
	0% {
    margin-left: 200px;
}

100% {
    margin-left: -200px;
}
}
@keyframes moveclouds2{
	0% {
    margin-right: 200px;
}
100% {
    margin-right: -200px;
}
}

/* ==================== Responsive Ipad ===================== */
@media (max-width: 991px) {
	    .coming-soon-content h6 {
        font-size: 14px;
    }
    .coming-soon-content h1 {
        font-size: 60px;
        padding-bottom: 20px;
    }
    .coming-soon-content form {
        height: 65px;
    }
}
/* ==================== Responsive Iphone ===================== */
@media (max-width: 460px) {
.creative-template .shape-four{
	display: none;
}
.creative-template .shape-three{
	display: none;
}
.coming-soon-content .row .col-md-9{
	width: 100%;
}
.coming-soon-content .row .col-md-9 p{
	    font-size: 20px;
    padding-bottom: 25px;
}
.coming-soon-content form input{
	    padding: 0 20px 0 20px;
}
.coming-soon-content form button{
	    width: 100%;
    position: relative;
    height: 65px;
    right: 0;
    top: 0;
    margin-top: 10px;
}
}

/* ====================== Buy Me A Coffee =============================== */

@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400..700&display=swap');
    .coffee-button {
      position: fixed;
      z-index:9;
      top: 20px;
      right: 20px;
      background-color: #FFC107;
      color: #333;
      padding: 8px 16px;
      border-radius: 20px;
      text-decoration: none;
      font-family: "Dancing Script", cursive;
      font-weight: 600;
      font-size: 20px;
      box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
      display: flex;
      align-items: center;
      gap: 8px;
      transition: background-color 0.3s;
    }

    .coffee-button:hover {
      background-color: #FFD54F;
    }

    .coffee-icon {
      font-size: 24px;
      line-height: 1;
    }
</style>