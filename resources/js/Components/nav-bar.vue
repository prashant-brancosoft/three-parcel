<script setup>
import { layoutMethods } from "@/state/helpers";
import { Link, router } from "@inertiajs/vue3";
import simplebar from "simplebar-vue";
import { ref, onMounted, computed, watch } from "vue";
import i18n from "../i18n";
import { initI18n, loadLocaleMessages } from "../i18n";
import { useSharedState } from "@/composables/useSharedState";
import store from "/resources/js/state/store.js";
import axios from "axios";
import { formatDistanceToNowStrict } from "date-fns";
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
    handleMarkAllAsRead,
} = useSharedState();

// Active menu tracking
const activeMainMenu = ref(null);
const activeCategory = ref(null);

// Search functionality
const searchQuery = ref("");
const showSearchResults = ref(false);
const searchInputRef = ref(null);

// Menu structure definition
const menuStructure = {
    dashboard: {
        label: "Dashboard",
        icon: "ri-home-4-line",
        route: "/dashboard",
        permission: "access-home",
    },
    chat: {
        label: "Chat",
        icon: "bx bx-message-rounded-dots",
        route: "/chat",
        permission: "chat",
    },
    orderManagement: {
        label: "Order Management",
        icon: "ri-shopping-cart-line",
        permission: "access-home",
        categories: {
            promotion: {
                label: "Promotion Management",
                icon: "ri-gift-line",
                permission: "promotion-management",
                items: [
                    {
                        label: "Promo Code",
                        route: "/promo-code",
                        permission: "manage-promo",
                        icon: "ri-coupon-line",
                    },
                    {
                        label: "Send Notification",
                        route: "/push-notifications",
                        permission: "view-notifications",
                        icon: "ri-notification-line",
                    },
                    {
                        label: "Banner Image",
                        route: "/banner-image",
                        permission: "banner_image",
                        icon: "ri-image-line",
                    },
                ],
            },
            price: {
                label: "Price Management",
                icon: "bx bx-money",
                permission: "price-management",
                items: [
                    {
                        label: "Service Location",
                        route: "/service-locations",
                        permission: "service-location",
                        icon: "ri-map-pin-line",
                    },
                    {
                        label: "Zone",
                        route: "/zones",
                        permission: "view-zone",
                        icon: "ri-global-line",
                    },
                    {
                        label: "Airport",
                        route: "/airport",
                        permission: "view-airport",
                        icon: "ri-plane-line",
                    },
                    {
                        label: "Vehicle Type",
                        route: "/vehicle_type",
                        permission: "vehicle-types",
                        icon: "ri-car-line",
                    },
                    {
                        label: "Rental Package",
                        route: "/rental-package-types",
                        permission: "rental-package",
                        icon: "ri-file-list-line",
                    },
                    {
                        label: "Set Price",
                        route: "/set-prices",
                        permission: "vehicle-fare",
                        icon: "ri-price-tag-3-line",
                    },
                    {
                        label: "Goods Type",
                        route: "/goods-type",
                        permission: "manage-goods-types",
                        icon: "ri-box-3-line",
                    },
                ],
            },
            geofencing: {
                label: "Geofencing",
                icon: "bx bx-map",
                permission: "geo-fencing",
                items: [
                    {
                        label: "Heat Map",
                        route: "/map/heat_map",
                        permission: "heat-map",
                        icon: "ri-fire-line",
                    },
                    {
                        label: "Gods Eye",
                        route: "/map/gods_eye",
                        permission: "gods-eye",
                        icon: "ri-eye-line",
                    },
                    {
                        label: "Peak Zone",
                        route: "/peak_zone",
                        permission: "peak-zone-view",
                        icon: "ri-map-2-line",
                    },
                ],
            },
            delivery: {
                // label: "Delivery & Rides",
                label: "Orders",
                icon: "bx bx-package",
                items: [
                    {
                        label: "Orders",
                        route: "/delivery-rides-request",
                        permission: "manage-delivery-request",
                        icon: "bx bx-package",
                    },
                    {
                        label: "Ongoing Orders",
                        route: "/ongoing-rides",
                        permission: "ongoing-request-view",
                        icon: "ri-taxi-fill",
                    },
                ],
            },
            others: {
                label: "Others",
                icon: "mdi mdi-dots-horizontal-circle",
                permission: "others",
                items: [
                    {
                        label: "SOS",
                        route: "/sos",
                        permission: "view-sos",
                        icon: "ri-alarm-warning-line",
                    },
                    {
                        label: "Cancellation",
                        route: "/cancellation",
                        permission: "view-cancellation",
                        icon: "ri-close-circle-line",
                    },
                    {
                        label: "FAQ",
                        route: "/faq",
                        permission: "view-faq",
                        icon: "ri-questionnaire-line",
                    },
                ],
            },
        },
    },
    users: {
        label: "Users",
        icon: "ri-user-3-line",
        permission: "access-user-nav-list",
        categories: {
            customer: {
                label: "Customer Management",
                icon: "ri-team-fill",
                permission: "user-management",
                items: [
                    {
                        label: "User List",
                        route: "/users",
                        permission: "view-users",
                        icon: "ri-user-line",
                    },
                    {
                        label: "Delete Request Users",
                        route: "/users/deleted-user",
                        permission: "view-delete-user",
                        icon: "ri-user-unfollow-line",
                    },
                    {
                        label: "User Bulk Upload",
                        route: "/user-import",
                        permission: "bulk_upload",
                        icon: "ri-upload-cloud-line",
                    },
                ],
            },
            driver: {
                label: "Driver Management",
                icon: "ri-user-follow-line",
                permission: "drivers-management",
                items: [
                    {
                        label: "Pending Drivers",
                        route: "/pending-drivers",
                        permission: "view-approval-pending-drivers",
                        icon: "ri-time-line",
                    },
                    {
                        label: "Approved Drivers",
                        route: "/approved-drivers",
                        permission: "view-approved-drivers",
                        icon: "ri-checkbox-circle-line",
                    },
                    {
                        label: "Subscription",
                        route: "/subscription",
                        permission: "manage-subscription",
                        icon: "ri-vip-crown-line",
                    },
                    {
                        label: "Drivers Ratings",
                        route: "/drivers-rating",
                        permission: "driver-rating-list",
                        icon: "ri-star-line",
                    },
                ],
            },
            admin: {
                label: "Admin Management",
                icon: "ri-group-line",
                permission: "admin",
                items: [
                    {
                        label: "Admins",
                        route: "/admins",
                        permission: "admin",
                        icon: "ri-admin-line",
                    },
                ],
            },
            referral: {
                label: "Referral",
                icon: "bx bx-share-alt",
                items: [
                    {
                        label: "Referral Settings",
                        route: "/referral-settings",
                        permission: "referral-settings-view",
                        icon: "bx bx-share-alt",
                    },
                ],
            },
            support: {
                label: "Issues Management",
                icon: "bx bx-support",
                permission: "view-support-management",
                items: [
                    {
                        label: "Issue Ticket Title",
                        route: "/title",
                        permission: "view-ticket-title",
                        icon: "ri-ticket-line",
                    },
                    {
                        label: "Issue Tickets",
                        route: "/support-tickets",
                        permission: "view-support-ticket",
                        icon: "ri-customer-service-line",
                    },
                ],
            },
            reports: {
                label: "Reports",
                icon: "ri-file-3-fill",
                permission: "report-management",
                items: [
                    {
                        label: "User Report",
                        route: "/report/user-report",
                        permission: "user-report",
                        icon: "ri-file-user-line",
                    },
                    {
                        label: "Driver Report",
                        route: "/report/driver-report",
                        permission: "driver-report",
                        icon: "ri-file-chart-line",
                    },
                ],
            },
        },
    },
    // settings: {
    //     label: "Settings",
    //     icon: "ri-settings-5-fill",
    //     permission: "access-settings-nav-list",
    //     categories: {
    //         business: {
    //             label: "Business Settings",
    //             icon: "ri-settings-5-fill",
    //             permission: "manage-business-settings",
    //             items: [
    //                 {
    //                     label: "General Settings",
    //                     route: "/general-settings",
    //                     permission: "general-settings-view",
    //                     icon: "ri-settings-3-line",
    //                 },
    //                 {
    //                     label: "Customization Settings",
    //                     route: "/customization-settings",
    //                     permission: "customization-settings-view",
    //                     icon: "ri-palette-line",
    //                 },
    //             ],
    //         },
    //         app: {
    //             label: "App Settings",
    //             icon: "mdi mdi-cellphone-cog",
    //             permission: "manage-app-settings",
    //             items: [
    //                 {
    //                     label: "Wallet Settings",
    //                     route: "/wallet-settings",
    //                     permission: "wallet-settings-view",
    //                     icon: "ri-wallet-line",
    //                 },
    //                 {
    //                     label: "Tip Settings",
    //                     route: "/tip-settings",
    //                     permission: "tip-settings-view",
    //                     icon: "ri-hand-coin-line",
    //                 },
    //             ],
    //         },
    //         thirdParty: {
    //             label: "Third Party Settings",
    //             icon: "mdi mdi-cogs",
    //             permission: "manage-third-party-settings",
    //             items: [
    //                 {
    //                     label: "Payment Gateway Settings",
    //                     route: "/payment-gateway",
    //                     permission: "payment-gateway-settings-view",
    //                     icon: "ri-bank-card-line",
    //                 },
    //                 {
    //                     label: "Firebase Settings",
    //                     route: "/firebase",
    //                     permission: "firebase-settings-view",
    //                     icon: "ri-firebase-line",
    //                 },
    //             ],
    //         },
    //     },
    // },
};

// Check if user has permission
const hasPermission = (permission) => {
    if (!permission) return true;
    const userPermissions = store.getters.permissions || [];
    return userPermissions.includes(permission);
};

// Get active menu and category based on current route
const getActiveMenuAndCategory = () => {
    const currentPath = window.location.pathname;

    for (const [menuKey, menu] of Object.entries(menuStructure)) {
        if (menu.route && currentPath.startsWith(menu.route)) {
            return { menu: menuKey, category: null };
        }

        if (menu.categories) {
            for (const [categoryKey, category] of Object.entries(
                menu.categories
            )) {
                if (category.items) {
                    for (const item of category.items) {
                        if (item.route && currentPath.startsWith(item.route)) {
                            return { menu: menuKey, category: categoryKey };
                        }
                    }
                }
            }
        }
    }

    return { menu: null, category: null };
};

// Handle category click
const handleCategoryClick = (categoryKey) => {
    if (activeCategory.value === categoryKey) {
        activeCategory.value = null;
    } else {
        activeCategory.value = categoryKey;
    }
};

// Search functionality
const allSearchItems = computed(() => {
    const items = [];

    Object.entries(menuStructure).forEach(([key, menu]) => {
        if (menu.route && hasPermission(menu.permission)) {
            items.push({
                label: menu.label,
                route: menu.route,
                icon: menu.icon,
                parent: null,
            });
        }

        if (menu.categories) {
            Object.entries(menu.categories).forEach(([catKey, category]) => {
                if (category.items && hasPermission(category.permission)) {
                    category.items.forEach((item) => {
                        if (hasPermission(item.permission)) {
                            items.push({
                                label: item.label,
                                route: item.route,
                                icon: item.icon,
                                parent: category.label,
                            });
                        }
                    });
                }
            });
        }
    });

    return items;
});

const filteredMenuItems = computed(() => {
    if (!searchQuery.value) return [];

    const query = searchQuery.value.toLowerCase();

    return allSearchItems.value
        .filter((item) => {
            const matchesSearch =
                item.label.toLowerCase().includes(query) ||
                (item.parent && item.parent.toLowerCase().includes(query));
            return matchesSearch;
        })
        .slice(0, 8);
});

const navigateToPage = (route) => {
    router.get(route);
    searchQuery.value = "";
    showSearchResults.value = false;
};

const handleClickOutside = (event) => {
    if (searchInputRef.value && !searchInputRef.value.contains(event.target)) {
        showSearchResults.value = false;
    }
};

const formatTimestamp = (timestamp) => {
    const date = new Date(timestamp);
    if (isNaN(date.getTime())) {
        return "Invalid date";
    }
    return formatDistanceToNowStrict(date, { addSuffix: true });
};

const selectedLabel = ref("");

onMounted(async () => {
    store.dispatch("fetchPermissions");
    await fetchData();
    await fetchLocations();
    await fetchFirebaseSettings();

    // Set active menu and category based on current route
    const active = getActiveMenuAndCategory();
    activeMainMenu.value = active.menu;
    activeCategory.value = active.category;

    document.addEventListener("click", handleClickOutside);

    var currentLocation = localStorage.getItem("selectedLocation");
    if (!currentLocation) {
        currentLocation = locations.value?.[0]?.value;
        localStorage.setItem("selectedLocation", currentLocation);
        setServiceLocation(currentLocation);
    }

    const serviceLocation = locations.value.find(
        (l) => l.value === currentLocation
    );
    selectedLabel.value = serviceLocation?.label;

    const currentLocale = localStorage.getItem("locale") || "en";
    selectedLanguageCode.value = currentLocale;
    const selectedLanguage = languages.value.find(
        (lang) => lang.code === currentLocale
    );

    if (selectedLanguage) {
        const direction = selectedLanguage.direction;
        const body = document.body;

        if (direction === "rtl") {
            localStorage.setItem("toggleDirection", true);
            body.classList.add("rtl");
            body.classList.remove("ltr");
        } else {
            localStorage.setItem("toggleDirection", false);
            body.classList.add("ltr");
            body.classList.remove("rtl");
        }
    }
});

const setLocation = async (location) => {
    setServiceLocation(location);
    const currentLocation = localStorage.getItem("selectedLocation");
    if (currentLocation) {
        const match = locations.value.find((l) => l.value === currentLocation);
        selectedLabel.value = match ? match.label : "";
        window.location.reload();
        localStorage.setItem("selectedLocation", match.value);
    }
};

const setLanguage = async (locale) => {
    await loadLocaleMessages(locale);
    i18n.global.locale = locale;
    selectedLanguageCode.value = locale;
    localStorage.setItem("locale", locale);

    const selectedLanguage = languages.value.find(
        (lang) => lang.code === locale
    );

    if (selectedLanguage) {
        const direction = selectedLanguage.direction;
        const body = document.body;

        if (direction === "rtl") {
            window.location.reload();
            localStorage.setItem("toggleDirection", true);
            body.classList.add("rtl");
            body.classList.remove("ltr");
        } else {
            window.location.reload();
            localStorage.setItem("toggleDirection", false);
            body.classList.add("ltr");
            body.classList.remove("rtl");
        }
    }
};

const handleChatClick = (chat_id) => {
    router.get("/chat?conversation_id=" + chat_id);
};

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
import { ref, onMounted } from "vue";
import simplebar from "simplebar-vue";
import i18n from "../i18n";
import { Link } from "@inertiajs/vue3";
import { mapGetters } from "vuex";

export default {
    data() {
        return {
            lan: i18n.global.locale,
            profilePhotoUrl: this.$page.props.auth.user.profile_picture,
        };
    },
    components: {
        simplebar,
        Link,
    },
    methods: {
        toggleHamburgerMenu() {
            const windowSize = document.documentElement.clientWidth;
            const layoutType =
                document.documentElement.getAttribute("data-layout");

            document.documentElement.setAttribute(
                "data-sidebar-visibility",
                "show"
            );
            const visibilityType = document.documentElement.getAttribute(
                "data-sidebar-visibility"
            );

            if (windowSize > 767)
                document
                    .querySelector(".hamburger-icon")
                    ?.classList.toggle("open");

            if (layoutType === "horizontal") {
                document.body.classList.toggle("menu");
            }

            if (
                visibilityType === "show" &&
                (layoutType === "vertical" || layoutType === "semibox")
            ) {
                if (windowSize < 1025 && windowSize > 767) {
                    document.body.classList.remove("vertical-sidebar-enable");
                    document.documentElement.setAttribute(
                        "data-sidebar-size",
                        document.documentElement.getAttribute(
                            "data-sidebar-size"
                        ) === "sm"
                            ? ""
                            : "sm"
                    );
                } else if (windowSize > 1025) {
                    document.body.classList.remove("vertical-sidebar-enable");
                    document.documentElement.setAttribute(
                        "data-sidebar-size",
                        document.documentElement.getAttribute(
                            "data-sidebar-size"
                        ) === "lg"
                            ? "sm"
                            : "lg"
                    );
                } else if (windowSize <= 767) {
                    document.body.classList.add("vertical-sidebar-enable");
                    document.documentElement.setAttribute(
                        "data-sidebar-size",
                        "lg"
                    );
                }
            }

            if (layoutType === "twocolumn") {
                document.body.classList.toggle("twocolumn-panel");
            }
        },

        initFullScreen() {
            document.body.classList.toggle("fullscreen-enable");
            if (
                !document.fullscreenElement &&
                !document.mozFullScreenElement &&
                !document.webkitFullscreenElement
            ) {
                if (document.documentElement.requestFullscreen) {
                    document.documentElement.requestFullscreen();
                } else if (document.documentElement.mozRequestFullScreen) {
                    document.documentElement.mozRequestFullScreen();
                } else if (document.documentElement.webkitRequestFullscreen) {
                    document.documentElement.webkitRequestFullscreen(
                        Element.ALLOW_KEYBOARD_INPUT
                    );
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
            const theme =
                document.documentElement.getAttribute("data-bs-theme") ===
                "dark"
                    ? "light"
                    : "dark";
            const sidebarColor =
                document.documentElement.getAttribute("data-sidebar") === "dark"
                    ? "light"
                    : "dark";
            document.documentElement.setAttribute("data-bs-theme", theme);
            document.documentElement.setAttribute("data-sidebar", sidebarColor);

            localStorage.setItem("toggleDarkMode", theme === "dark");
        },

        savedToggleTheme() {
            const isDarkMode =
                localStorage.getItem("toggleDarkMode") === "true";
            const theme = isDarkMode ? "dark" : "light";
            const sidebarColor = isDarkMode ? "dark" : "light";

            document.documentElement.setAttribute("data-bs-theme", theme);
            document.documentElement.setAttribute("data-sidebar", sidebarColor);
        },
    },
    computed: {
        ...mapGetters(["permissions"]),

        supportTicket() {
            return window.supportTicket || 0;
        },
    },
    mounted() {
        this.savedToggleTheme();
        document.addEventListener("scroll", function () {
            const pageTopbar = document.getElementById("page-topbar");
            if (pageTopbar) {
                document.body.scrollTop >= 50 ||
                document.documentElement.scrollTop >= 50
                    ? pageTopbar.classList.add("topbar-shadow")
                    : pageTopbar.classList.remove("topbar-shadow");
            }
        });
        if (document.getElementById("topnav-hamburger-icon")) {
            document
                .getElementById("topnav-hamburger-icon")
                .addEventListener("click", this.toggleHamburgerMenu);
        }
    },
};
</script>

<template>
    <header id="page-topbar">
        <!-- Row 1: Main Navigation -->
        <div class="layout-width">
            <div class="navbar-main-row">
                <!-- Left: Main Menu Items -->
                <div class="main-menu-container">
                    <nav class="navbar navbar-expand">
                        <ul class="navbar-nav">
                            <!-- Dashboard -->
                            <li
                                class="nav-item"
                                v-if="
                                    hasPermission(
                                        menuStructure.dashboard.permission
                                    )
                                "
                            >
                                <Link
                                    :href="menuStructure.dashboard.route"
                                    class="nav-link"
                                    :class="{
                                        active: activeMainMenu === 'dashboard',
                                    }"
                                >
                                    <i
                                        :class="menuStructure.dashboard.icon"
                                    ></i>
                                    <span>{{
                                        menuStructure.dashboard.label
                                    }}</span>
                                </Link>
                            </li>

                            <!-- Chat -->
                            <li
                                class="nav-item"
                                v-if="
                                    hasPermission(menuStructure.chat.permission)
                                "
                            >
                                <Link
                                    :href="menuStructure.chat.route"
                                    class="nav-link"
                                    :class="{
                                        active: activeMainMenu === 'chat',
                                    }"
                                >
                                    <i :class="menuStructure.chat.icon"></i>
                                    <span>{{ menuStructure.chat.label }}</span>
                                </Link>
                            </li>

                            <!-- Order Management -->
                            <li
                                class="nav-item"
                                v-if="
                                    hasPermission(
                                        menuStructure.orderManagement.permission
                                    )
                                "
                            >
                                <a
                                    href="javascript:void(0)"
                                    class="nav-link"
                                    :class="{
                                        active:
                                            activeMainMenu ===
                                            'orderManagement',
                                    }"
                                    @click="
                                        activeMainMenu =
                                            activeMainMenu === 'orderManagement'
                                                ? null
                                                : 'orderManagement';
                                        activeCategory = null;
                                    "
                                >
                                    <i
                                        :class="
                                            menuStructure.orderManagement.icon
                                        "
                                    ></i>
                                    <span>{{
                                        menuStructure.orderManagement.label
                                    }}</span>
                                </a>
                            </li>

                            <!-- Users -->
                            <li
                                class="nav-item"
                                v-if="
                                    hasPermission(
                                        menuStructure.users.permission
                                    )
                                "
                            >
                                <a
                                    href="javascript:void(0)"
                                    class="nav-link"
                                    :class="{
                                        active: activeMainMenu === 'users',
                                    }"
                                    @click="
                                        activeMainMenu =
                                            activeMainMenu === 'users'
                                                ? null
                                                : 'users';
                                        activeCategory = null;
                                    "
                                >
                                    <i :class="menuStructure.users.icon"></i>
                                    <span>{{ menuStructure.users.label }}</span>
                                </a>
                            </li>

                            <!-- Settings -->
                            <!-- <li
                                class="nav-item"
                                v-if="
                                    hasPermission(
                                        menuStructure.settings.permission
                                    )
                                "
                            >
                                <a
                                    href="javascript:void(0)"
                                    class="nav-link"
                                    :class="{
                                        active: activeMainMenu === 'settings',
                                    }"
                                    @click="
                                        activeMainMenu =
                                            activeMainMenu === 'settings'
                                                ? null
                                                : 'settings';
                                        activeCategory = null;
                                    "
                                >
                                    <i :class="menuStructure.settings.icon"></i>
                                    <span>{{
                                        menuStructure.settings.label
                                    }}</span>
                                </a>
                            </li> -->
                            <!-- support Management -->
                            <!-- <li data-v-2ad04a9c="" class="nav-item">
                                <a
                                    data-v-2ad04a9c=""
                                    href="javascript:void(0)"
                                    class="nav-link active"
                                    ><i
                                        data-v-2ad04a9c=""
                                        class="ri-user-3-line"
                                    ></i
                                    ><span data-v-2ad04a9c=""
                                        >Support Management</span
                                    ></a
                                >
                            </li> -->
                            <li data-v-2ad04a9c="" class="nav-item">
                                <a
                                    data-v-2ad04a9c=""
                                    href="/support-tickets"
                                    class="nav-link"
                                >
                                    <i
                                        data-v-2ad04a9c=""
                                        class="ri-chat-1-line"
                                    ></i>
                                    <span data-v-2ad04a9c=""
                                        >Issue Tickets</span
                                    >
                                    <span
                                        v-if="supportTicket > 0"
                                        class="badge bg-danger rounded-pill ms-1"
                                        >{{ supportTicket }}</span
                                    >
                                </a>
                            </li>
                        </ul>
                    </nav>
                </div>

                <!-- Right: Search and Profile -->
                <div class="d-flex align-items-center">
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
                            />
                        </div>

                        <!-- Search Results -->
                        <div
                            v-if="showSearchResults && searchQuery"
                            class="search-results"
                        >
                            <div
                                v-if="filteredMenuItems.length > 0"
                                class="results-list"
                            >
                                <div
                                    v-for="item in filteredMenuItems"
                                    :key="item.route"
                                    class="search-result-item"
                                    @click="navigateToPage(item.route)"
                                >
                                    <i
                                        :class="item.icon"
                                        class="result-icon"
                                    ></i>
                                    <div class="result-content">
                                        <div class="result-name">
                                            {{ item.label }}
                                        </div>
                                        <div
                                            v-if="item.parent"
                                            class="result-parent"
                                        >
                                            {{ item.parent }}
                                        </div>
                                    </div>
                                    <i
                                        class="ri-arrow-right-s-line result-arrow"
                                    ></i>
                                </div>
                            </div>
                            <div v-else class="no-results">
                                <i class="ri-search-line"></i>
                                <p>No results found</p>
                            </div>
                        </div>
                    </div>

                    <!-- Profile Dropdown -->
                    <BDropdown
                        variant="link"
                        class="header-item topbar-user"
                        toggle-class="rounded-circle arrow-none"
                        menu-class="dropdown-menu-end"
                        :offset="{
                            alignmentAxis: -14,
                            crossAxis: 0,
                            mainAxis: 0,
                        }"
                    >
                        <template #button-content>
                            <span class="d-flex align-items-center">
                                <img
                                    v-if="
                                        $page.props.jetstream
                                            .managesProfilePhotos
                                    "
                                    class="rounded-circle header-profile-user"
                                    :src="
                                        profilePhotoUrl ||
                                        '/default-profile.jpeg'
                                    "
                                    :alt="$page.props.auth.user.name"
                                />
                                <span class="text-start ms-xl-2">
                                    <span
                                        class="d-none d-xl-inline-block ms-1 fw-medium user-name-text"
                                    >
                                        {{ $page.props.auth.user.name }}
                                    </span>
                                </span>
                            </span>
                        </template>
                        <h6 class="dropdown-header">
                            {{ $t("welcome") }}
                            {{ $page.props.auth.user.name }}!
                        </h6>
                        <Link class="dropdown-item" href="/profile-edit">
                            <i
                                class="mdi mdi-account-circle text-muted fs-16 align-middle me-1"
                            ></i>
                            <span class="align-middle">{{
                                $t("profile")
                            }}</span>
                        </Link>
                        <form
                            method="POST"
                            @submit.prevent="logout"
                            class="dropdown-item"
                        >
                            <BButton
                                variant="none"
                                type="submit"
                                class="btn p-0"
                            >
                                <i
                                    class="mdi mdi-logout text-muted fs-16 align-middle me-1"
                                ></i>
                                {{ $t("logout") }}
                            </BButton>
                        </form>
                    </BDropdown>
                </div>
            </div>
        </div>

        <!-- Row 2: Categories (when main menu is active) -->
        <transition name="submenu-slide">
            <div
                v-if="
                    activeMainMenu && menuStructure[activeMainMenu]?.categories
                "
                class="categories-row"
            >
                <div class="layout-width">
                    <div class="categories-container">
                        <button
                            v-for="(category, categoryKey) in menuStructure[
                                activeMainMenu
                            ].categories"
                            :key="categoryKey"
                            v-show="hasPermission(category.permission)"
                            class="category-button"
                            :class="{ active: activeCategory === categoryKey }"
                            @click="handleCategoryClick(categoryKey)"
                        >
                            <i :class="category.icon"></i>
                            <span>{{ category.label }}</span>
                        </button>
                    </div>
                </div>
            </div>
        </transition>

        <!-- Row 3: Menu Items (when category is active) -->
        <transition name="items-slide">
            <div
                v-if="
                    activeCategory &&
                    menuStructure[activeMainMenu]?.categories[activeCategory]
                "
                class="items-row"
            >
                <div class="layout-width">
                    <div class="items-container">
                        <Link
                            v-for="item in menuStructure[activeMainMenu]
                                .categories[activeCategory].items"
                            :key="item.route"
                            v-show="hasPermission(item.permission)"
                            :href="item.route"
                            class="menu-item"
                            :class="{
                                active: $page.url.startsWith(item.route),
                            }"
                        >
                            <i :class="item.icon"></i>
                            <span>{{ item.label }}</span>
                        </Link>
                    </div>
                </div>
            </div>
        </transition>
    </header>
</template>

<style scoped>
/* Main Header */
#page-topbar {
    position: sticky;
    top: 0;
    z-index: 1000;
    background: #4a5a92;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Row 1: Main Navigation */
.navbar-main-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 1rem;
    min-height: 60px;
}

.main-menu-container .navbar-nav {
    display: flex;
    gap: 0.25rem;
    align-items: center;
    list-style: none;
    margin: 0;
    padding: 0;
}

.nav-item .nav-link {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.75rem 1rem;
    color: rgba(255, 255, 255, 0.85);
    font-size: 12px;
    font-weight: 500;
    text-decoration: none;
    border-radius: 4px;
    transition: all 0.2s ease;
    cursor: pointer;
    background: transparent;
    border: none;
    white-space: nowrap;
}

.nav-item .nav-link:hover {
    background: rgba(255, 255, 255, 0.1);
    color: #fff;
}

.nav-item .nav-link.active {
    background: rgba(255, 255, 255, 0.2);
    color: #fff;
    font-weight: 600;
}

.nav-link i {
    font-size: 18px;
}

/* Row 2: Categories */
.categories-row {
    background: #f8f9fa;
    border-bottom: 2px solid #e9ecef;
    padding: 0.25rem 0;
}

.categories-container {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
    padding: 0 1rem;
}

.category-button {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.65rem 1.25rem;
    background: #fff;
    border: 2px solid #e9ecef;
    border-radius: 6px;
    color: #495057;
    font-size: 11px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.3px;
    cursor: pointer;
    transition: all 0.2s ease;
    white-space: nowrap;
}

.category-button:hover {
    background: #e9ecef;
    border-color: #405189;
    color: #405189;
}

.category-button.active {
    background: #405189;
    border-color: #405189;
    color: #fff;
    box-shadow: 0 2px 6px rgba(64, 81, 137, 0.3);
}

.category-button i {
    font-size: 16px;
}

/* Row 3: Menu Items */
.items-row {
    background: #fff;
    border-bottom: 1px solid #e9ecef;
    padding: 0.25rem 0;
}

.items-container {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
    padding: 0 1rem;
}

.menu-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.6rem 1rem;
    background: #f8f9fa;
    border: 1px solid #e9ecef;
    border-radius: 4px;
    color: #495057;
    font-size: 12px;
    font-weight: 500;
    text-decoration: none;
    transition: all 0.2s ease;
    white-space: nowrap;
}

.menu-item:hover {
    background: #e9ecef;
    color: #405189;
    border-color: #405189;
}

.menu-item.active {
    background: #d4e4ff;
    color: #405189;
    border-color: #405189;
    font-weight: 600;
    box-shadow: 0 2px 4px rgba(64, 81, 137, 0.2);
}

.menu-item i {
    font-size: 16px;
}

/* Search Container */
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
    color: rgba(255, 255, 255, 0.6);
    font-size: 18px;
    pointer-events: none;
}

.search-input {
    padding-left: 40px;
    height: 38px;
    border-radius: 6px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    background: rgba(255, 255, 255, 0.1);
    color: #fff;
    font-size: 12px;
    transition: all 0.3s ease;
}

.search-input::placeholder {
    color: rgba(255, 255, 255, 0.6);
}

.search-input:focus {
    background: rgba(255, 255, 255, 0.15);
    border-color: rgba(255, 255, 255, 0.3);
    box-shadow: 0 0 0 0.2rem rgba(255, 255, 255, 0.1);
    outline: none;
}

/* Search Results */
.search-results {
    position: absolute;
    top: calc(100% + 8px);
    left: 0;
    right: 0;
    background: #fff;
    border: 1px solid #e9ecef;
    border-radius: 8px;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
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
    font-size: 12px;
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
    font-size: 12px;
}

/* Profile */
.header-profile-user {
    width: 36px;
    height: 36px;
    object-fit: cover;
}

.user-name-text {
    color: rgba(255, 255, 255, 0.9);
    font-size: 12px;
}

/* Transitions */
.submenu-slide-enter-active,
.submenu-slide-leave-active,
.items-slide-enter-active,
.items-slide-leave-active {
    transition: all 0.3s ease;
}

.submenu-slide-enter-from,
.items-slide-enter-from {
    opacity: 0;
    max-height: 0;
    transform: translateY(-10px);
}

.submenu-slide-leave-to,
.items-slide-leave-to {
    opacity: 0;
    max-height: 0;
    transform: translateY(-10px);
}

/* Layout */
.layout-width {
    max-width: 100%;
    margin: 0 auto;
}

/* Responsive Design */
@media screen and (max-width: 1200px) {
    .categories-container,
    .items-container {
        gap: 0.4rem;
    }

    .category-button {
        padding: 0.6rem 1rem;
        font-size: 12px;
    }

    .menu-item {
        padding: 0.55rem 0.9rem;
        font-size: 13px;
    }
}

@media screen and (max-width: 992px) {
    .nav-link span {
        display: none;
    }

    .nav-link {
        padding: 0.75rem !important;
    }

    .search-container {
        width: 200px;
    }

    .user-name-text {
        display: none !important;
    }

    .category-button span {
        display: none;
    }

    .category-button {
        padding: 0.6rem;
    }
}

@media screen and (max-width: 768px) {
    .navbar-main-row {
        flex-wrap: wrap;
        padding: 0.5rem 1rem;
    }

    .main-menu-container {
        width: 100%;
        order: 2;
        margin-top: 0.5rem;
    }

    .main-menu-container .navbar-nav {
        width: 100%;
        overflow-x: auto;
        scrollbar-width: thin;
        padding-bottom: 0.5rem;
    }

    .main-menu-container .navbar-nav::-webkit-scrollbar {
        height: 3px;
    }

    .main-menu-container .navbar-nav::-webkit-scrollbar-thumb {
        background: rgba(255, 255, 255, 0.3);
        border-radius: 3px;
    }

    .search-container {
        width: 180px;
    }

    .categories-container,
    .items-container {
        overflow-x: auto;
        flex-wrap: nowrap;
        scrollbar-width: thin;
    }

    .categories-container::-webkit-scrollbar,
    .items-container::-webkit-scrollbar {
        height: 3px;
    }

    .categories-container::-webkit-scrollbar-thumb,
    .items-container::-webkit-scrollbar-thumb {
        background: #cbd5e1;
        border-radius: 3px;
    }
}

@media screen and (max-width: 576px) {
    .search-container {
        width: 150px;
    }

    .search-input {
        font-size: 13px;
        height: 36px;
    }

    .nav-link {
        padding: 0.6rem 0.75rem !important;
    }

    .nav-link i {
        font-size: 16px;
    }

    .menu-item span {
        display: none;
    }

    .menu-item {
        padding: 0.6rem;
    }
}

/* Dark Mode Support */
[data-bs-theme="dark"] #page-topbar {
    background: #1e293b;
}

[data-bs-theme="dark"] .categories-row {
    background: #0f172a;
    border-bottom-color: #334155;
}

[data-bs-theme="dark"] .category-button {
    background: #1e293b;
    border-color: #334155;
    color: #cbd5e1;
}

[data-bs-theme="dark"] .category-button:hover {
    background: #334155;
    border-color: #60a5fa;
    color: #60a5fa;
}

[data-bs-theme="dark"] .category-button.active {
    background: #1e3a5f;
    border-color: #60a5fa;
    color: #fff;
}

[data-bs-theme="dark"] .items-row {
    background: #0f172a;
    border-bottom-color: #334155;
}

[data-bs-theme="dark"] .menu-item {
    background: #1e293b;
    border-color: #334155;
    color: #cbd5e1;
}

[data-bs-theme="dark"] .menu-item:hover {
    background: #334155;
    color: #60a5fa;
    border-color: #60a5fa;
}

[data-bs-theme="dark"] .menu-item.active {
    background: #1e3a5f;
    color: #60a5fa;
    border-color: #60a5fa;
}

[data-bs-theme="dark"] .search-results {
    background: #1e293b;
    border-color: #334155;
}

[data-bs-theme="dark"] .search-result-item:hover {
    background: #334155;
}

[data-bs-theme="dark"] .result-name {
    color: #cbd5e1;
}

/* Scrollbar Styling */
.search-results::-webkit-scrollbar {
    width: 6px;
}

.search-results::-webkit-scrollbar-track {
    background: #f1f5f9;
}

.search-results::-webkit-scrollbar-thumb {
    background: #cbd5e1;
    border-radius: 3px;
}

.search-results::-webkit-scrollbar-thumb:hover {
    background: #94a3b8;
}

/* Profile Dropdown Fix */
.topbar-user {
    margin-left: 0.5rem;
}

.topbar-user .btn-link {
    color: inherit;
    text-decoration: none;
    padding: 0;
}

.topbar-user .btn-link:hover,
.topbar-user .btn-link:focus {
    color: inherit;
    text-decoration: none;
}

/* Better visual hierarchy */
.navbar-nav .nav-item {
    margin: 0;
}

.nav-link {
    position: relative;
}

.nav-link.active::after {
    content: "";
    position: absolute;
    bottom: -2px;
    left: 10%;
    right: 10%;
    height: 3px;
    background: #fff;
    border-radius: 3px 3px 0 0;
}
</style>
