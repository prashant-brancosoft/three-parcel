<script>
import { Link, Head, useForm, router } from '@inertiajs/vue3';
import Layout from "@/Layouts/main.vue";
import PageHeader from "@/Components/page-header.vue";
import Pagination from "@/Components/Pagination.vue";
import Swal from "sweetalert2";
import { ref, onMounted, watch, computed } from "vue";
import axios from "axios";
import { debounce } from 'lodash';
import Multiselect from "@vueform/multiselect";
import "@vueform/multiselect/themes/default.css";
import flatPickr from "vue-flatpickr-component";
import "flatpickr/dist/flatpickr.css";
import useVuelidate from "@vuelidate/core";
import { UsersIcon } from '@zhuowenli/vue-feather-icons';
import getChartColorsArray from "@/common/getChartColorsArray";
import { useSharedState } from '@/composables/useSharedState';
import { useI18n } from 'vue-i18n';
import { layoutComputed } from "@/state/helpers";
import { mapGetters } from 'vuex';
import Warning from "@/Components/warning.vue";

export default {
  props: {
    firebaseConfig: Object,

  },
  data() {
    return {
      selectedServiceLocation: null, // To store the selected service location
    };
  },
  computed: {
    ...layoutComputed,
    ...mapGetters(['permissions']),
    layoutType: {
      get() {
        return this.$store ? this.$store.state.layout.layoutType : {} || {};
      },
    },
  },
  components: {
    Layout,
    PageHeader,
    Head,
    Pagination,
    Multiselect,
    flatPickr,
    Link,
    UsersIcon,
    Warning
  },
  setup(props) {
    const { t } = useI18n();
    const { playAudioOnce, selectedLocation } = useSharedState();
    const series = ref([]);
    const chartOptions = ref({});
    const overall = ref([]);
    const overallChartOptions = ref({});
    const cancellation = ref([]);
    const cancelChartOptions = ref({});
    const sosRequests = ref([]);
    const seriesOverallTrip = ref([]);
    const cancelledtrips = ref({
      auto_cancelled : 0,
      user_cancelled : 0,
      driver_cancelled : 0,
      dispatcher_cancelled : 0,
      total_cancelled : 0,
    });
    const chartOptionsOverallTrip = ref({});
    const earningData = ref({
        card : 0,
        cash : 0,
        wallet : 0,
        total : 0,
        admin_commision : 0,
        driver_commision : 0,
    });

    const todayEarnings = ref(earningData.value);
    const overallEarnings = ref(earningData.value);

    const totalDrivers = ref({
        approved : 0,
        declined : 0,
        approve_percentage : 0,
        decline_percentage : 0,
        total : 0,
    });
    const totalUsers = ref(0);
    const currencySymbol = ref('');

    const fetchDashboardData = async () => {
      try {
        const response = await axios.get('/dashboard/data',{ params:{service_location_id : selectedLocation.value}});
        totalDrivers.value = response.data.totalDrivers;
        totalUsers.value = response.data.totalUsers;
        currencySymbol.value = response.data.currencySymbol;

      } catch (error) {
        console.error(t('error_fetching_today_earnings'), error);
      }
    }

    // Fetch data for today earnings chart
    const fetchTodayEarnings = async () => {
      try {
        const response = await axios.get('/dashboard/today-earnings',{ params:{service_location_id : selectedLocation.value}});
        series.value = [
          Number(response.data.today.completed),
          Number(response.data.today.cancelled),
          Number(response.data.today.scheduled),
        ];

        todayEarnings.value = response.data.today.earnings;
        chartOptions.value = {
          labels: [t('completed'), t('cancelled'), t('scheduled')],
          chart: {
            type: "donut",
            height: 219,
          },
          plotOptions: {
            pie: {
              size: 100,
              donut: {
                size: "75%",
              },
            },
          },
          dataLabels: {
            enabled: false,
          },
          legend: {
            show: false,
            position: "bottom",
            horizontalAlign: "center",
            offsetX: 0,
            offsetY: 0,
            markers: {
              width: 20,
              height: 6,
              radius: 2,
            },
            itemMargin: {
              horizontal: 12,
              vertical: 0,
            },
          },
          stroke: {
            width: 0,
          },
          yaxis: {
            labels: {
              formatter: function (value) {
                return value;
              },
            },
            tickAmount: 4,
            min: 0,
          },
          colors: getChartColorsArray('["--vz-primary", "--vz-warning", "--vz-info"]'),
        };

        seriesOverallTrip.value = [
          Number(response.data.overall.completed),
          Number(response.data.overall.cancelled),
          Number(response.data.overall.scheduled),
        ];
        overallEarnings.value = response.data.overall.earnings;
        chartOptionsOverallTrip.value = generateChartOptions('overall');
      } catch (error) {
        console.error(t('error_fetching_today_earnings'), error);
      }
    };


    // Helper function to generate chart options
    const generateChartOptions = (type) => {
      return {
        labels: [t('completed'), t('cancelled'), t('scheduled')],
        chart: {
          type: 'donut',
          height: 219,
        },
        plotOptions: {
          pie: {
            size: 100,
            donut: {
              size: '75%',
            },
          },
        },
        dataLabels: {
          enabled: false,
        },
        legend: {
          show: false,
          position: 'bottom',
          horizontalAlign: 'center',
          markers: {
            width: 20,
            height: 6,
            radius: 2,
          },
        },
        stroke: {
          width: 0,
        },
        yaxis: {
          labels: {
            formatter: function (value) {
              return value;
            },
          },
          tickAmount: 4,
          min: 0,
        },
        colors: getChartColorsArray(
          type === 'today'
            ? '["--vz-primary", "--vz-warning", "--vz-info"]'
            : '["--vz-secondary", "--vz-danger", "--vz-success"]'
        ),
      };
    };

    const sos_update = async(sos) => {
      try {

        const response = await axios.get(`rides-request/detail/${sos.req_id}`);
        if (response.status === 200) {
          let trip = response.data.request;
          let sosData = {
            isUser: sos.is_user,
            isDriver: sos.is_driver,
            userName: trip.userDetail?.data?.name,
            driverName: trip.driverDetail?.data?.name,
            request_id: sos.req_id,
            date: response.data.current_time,
          };
          const existingIndex = sosRequests.value.findIndex(
            (request) => request.request_id === sosData.request_id
          );
          if (existingIndex === -1) {
            sosRequests.value.push(sosData);
          } else {
            sosRequests.value[existingIndex] = sosData;
          }
          playAudioOnce();
            Swal.fire({
              title: t('notified_sos'),
              text: t('sos_has_been_notified_proceed_to_details'),
              icon: "warning",
              showCancelButton: true,
              confirmButtonColor: "#34c38f",
              cancelButtonColor: "#f46a6a",
              confirmButtonText: t('check'),
              cancelButtonText: t('cancel')
          }).then(async (result) => {
              if (result.isConfirmed) {
                  router.get('/rides-request/view/'+sos.req_id);
              }
          });
        }
      }catch (error) {
        console.error(error);
      }
    }


    // Fetch data for overall earnings chart
    const fetchOverallEarnings = async () => {
      try {
        const response = await axios.get('/dashboard/overall-earnings',{ params:{service_location_id : selectedLocation.value}});
        overall.value = [
          {
            name: t('overall_earnings'),
            data: response.data.earnings.values,
          },
        ];
        overallChartOptions.value = {
          chart: {
            height: 100,
            type: "area",
            toolbar: "false",
          },
          dataLabels: {
            enabled: false,
          },
          stroke: {
            curve: "smooth",
            width: 3,
          },
          xaxis: {
            categories: response.data.earnings.months, // x Axis months
          },
          yaxis: {
            labels: {
              formatter: function (value) {
                return value.toFixed(1);
              },
            },
            tickAmount: 5,
            min: 0,
            max: Math.max(...response.data.earnings.values) * 1.1, // Adjust max value dynamically
          },
          colors: getChartColorsArray('["--vz-success"]'),
          fill: {
            opacity: 0,
            colors: ["#0AB39C", "#F06548"],
            type: "solid",
          },
        };
      } catch (error) {
        console.error(t('error_fetching_overall_earnings'), error);
      }
    };

    // Fetch data for cancellation chart
    const fetchCancellationData = async () => {
      try {
        const response = await axios.get('/dashboard/cancel-chart',{ params:{service_location_id : selectedLocation.value}});
        cancelledtrips.value = response.data['data'];
        cancellation.value = [
          {
            name: t('cancelled_due_to_no_drivers'),
            type: "bar",
            data: response.data['a'],
          },
          {
            name: t('cancelled_by_users'),
            type: "bar",
            data: response.data['u'],
          },
          {
            name: t('cancelled_by_drivers'),
            type: "bar",
            data: response.data['d'],
          },
        ];
        cancelChartOptions.value = {
          chart: {
            height: 374,
            type: "line",
            toolbar: {
              show: false,
            },
          },
          stroke: {
            curve: "smooth",
            dashArray: [0, 0, 0],
            width: [0, 0, 0],
          },
          fill: {
            opacity: [1, 1, 1],
          },
          markers: {
            size: [0, 0, 0],
            strokeWidth: 2,
            hover: {
              size: 4,
            },
          },
          xaxis: {
            categories: response.data['y'],
            axisTicks: {
              show: false,
            },
            axisBorder: {
              show: false,
            },
          },
          grid: {
            show: true,
            xaxis: {
              lines: {
                show: true,
              },
            },
            yaxis: {
              lines: {
                show: false,
              },
            },
            padding: {
              top: 0,
              right: -2,
              bottom: 15,
              left: 10,
            },
          },
          legend: {
            show: true,
            horizontalAlign: "center",
            offsetX: 0,
            offsetY: -5,
            markers: {
              width: 9,
              height: 9,
              radius: 6,
            },
            itemMargin: {
              horizontal: 10,
              vertical: 0,
            },
          },
          plotOptions: {
            bar: {
              columnWidth: "30%",
              barHeight: "70%",
            },
          },
          colors: getChartColorsArray('["--vz-primary", "--vz-warning", "--vz-success"]'),
          tooltip: {
            shared: true,
            y: [
              {
                formatter: function (y) {
                  if (typeof y !== "undefined") {
                    return y.toFixed(0);
                  }
                  return y;
                },
              },
              {
                formatter: function (y) {
                  if (typeof y !== "undefined") {
                    return y.toFixed(0);
                  }
                  return y;
                },
              },
              {
                formatter: function (y) {
                  if (typeof y !== "undefined") {
                    return y.toFixed(0);
                  }
                  return y;
                },
              },
            ],
          },
        };
      } catch (error) {
        console.error(t('error_fetching_cancellation_data'), error);
      }
    };

    const fetchAllData = async() => {
        await fetchDashboardData();
        await fetchTodayEarnings();
        await fetchOverallEarnings();
        await fetchCancellationData();
    }
    // Call APIs on component mount
    onMounted(async() => {
      try{
        let shouldProcessSosChildAdded = false;
        setTimeout(()=> {
          shouldProcessSosChildAdded = true;
        },4000);
        await fetchAllData();
        const firebaseConfig = props.firebaseConfig;
        if (!firebase.apps.length) {
          firebase.initializeApp(firebaseConfig);
        }
        const sosRef = firebase.database().ref('SOS');
        
        sosRef.on('child_changed', async function(snapshot) {
          var sosData = snapshot.val();
          if (shouldProcessSosChildAdded)
          {
            await sos_update(sosData);
          } 
        });
        sosRef.on('child_added', async function(snapshot) {
            var sosData = snapshot.val();
            if (shouldProcessSosChildAdded)
            {
                await sos_update(sosData);
            } 
        });
      }catch (error) {
        console.error(error);
      }
    });

    watch (()=>selectedLocation.value, (value) => {
      if(value){
          fetchAllData();
      }
    })

    return {
      series,
      chartOptions,
      overall,
      sosRequests,
      overallChartOptions,
      cancellation,
      cancelChartOptions,
      seriesOverallTrip,
      chartOptionsOverallTrip,
      todayEarnings,
      overallEarnings,
      totalDrivers,
      totalUsers,
      cancelledtrips,
      currencySymbol,
    };
  },

  methods: {},
};
</script>


<template>
  <Layout>
    
    <Warning />
    <PageHeader :title="$t('dashboard')" :pageTitle="$t('dashboard')" />
    <div class="container-fluid dashboard-sections">
        <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 row-cols-xl-3 g-4">
           

            <!-- Order Management Card -->
            <div class="col">
                <div class="card h-100">
                    <div class="card-header"><i class="fas fa-shopping-cart me-2"></i>Order Management</div>
                    <ul class="list-group list-group-flush">
                        <li class="list-group-item">
                            <a href="#"><i class="fas fa-tags"></i>Promotion Management</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/promo-code"><i class="fas fa-ticket-alt"></i>Promo Code</a></li>
                                <li class="list-group-item"><a href="/push-notifications"><i class="fas fa-bell"></i>Send Notification</a></li>
                                <li class="list-group-item"><a href="/banner-image"><i class="fas fa-image"></i>Banner Image</a></li>
                            </ul>
                        </li>
                        <li class="list-group-item">
                            <a href="#"><i class="fas fa-dollar-sign"></i>Price Management</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/service-locations"><i class="fas fa-map-marker-alt"></i>Service Location</a></li>
                                <li class="list-group-item"><a href="/zones"><i class="fas fa-globe"></i>Zone</a></li>
                                <li class="list-group-item"><a href="/airport"><i class="fas fa-plane"></i>Airport</a></li>
                                <li class="list-group-item"><a href="/vehicle_type"><i class="fas fa-car"></i>Vehicle Type</a></li>
                                <!-- <li class="list-group-item"><a href="/rental-package-types"><i class="fas fa-box"></i>Rental Package</a></li> -->
                                <li class="list-group-item"><a href="/set-prices"><i class="fas fa-money-bill"></i>Set Price</a></li>
                                <li class="list-group-item"><a href="/goods-type"><i class="fas fa-boxes"></i>Goods Type</a></li>
                            </ul>
                        </li>
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-map"></i>Geofencing</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/map/heat_map"><i class="fas fa-fire"></i>Heat Map</a></li>
                                <li class="list-group-item"><a href="/map/gods_eye"><i class="fas fa-eye"></i>God's Eye</a></li>
                                <li class="list-group-item"><a href="/peak_zone"><i class="fas fa-map-pin"></i>Peak Zone</a></li>
                            </ul>
                        </li> -->
                        <!-- <li class="list-group-item"><a href="/delivery-rides-request"><i class="fas fa-truck"></i>Delivery Requests</a></li>
                        <li class="list-group-item"><a href="/ongoing-rides"><i class="fas fa-road"></i>Ongoing Rides</a></li>
                        <li class="list-group-item">
                            <a href="#"><i class="fas fa-ellipsis-h"></i>Others</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/sos"><i class="fas fa-exclamation-circle"></i>SOS</a></li>
                                <li class="list-group-item"><a href="/cancellation"><i class="fas fa-ban"></i>Cancellation</a></li>
                                <li class="list-group-item"><a href="/faq"><i class="fas fa-question-circle"></i>FAQ</a></li>
                            </ul>
                        </li> -->
                    </ul>
                </div>
            </div>

            <!-- Users Card -->
            <div class="col">
                <div class="card h-100">
                    <div class="card-header"><i class="fas fa-users me-2"></i>Users</div>
                    <ul class="list-group list-group-flush">
                        <li class="list-group-item">
                            <a href="#"><i class="fas fa-user-friends"></i>Customer Management</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/users"><i class="fas fa-list"></i>User List</a></li>
                                <li class="list-group-item"><a href="/users/deleted-user"><i class="fas fa-user-slash"></i>Delete Request Users</a></li>
                                <li class="list-group-item"><a href="/user-import"><i class="fas fa-upload"></i>User Bulk Upload</a></li>
                            </ul>
                        </li>
                        <li class="list-group-item">
                            <a href="#"><i class="fas fa-user-tie"></i>Driver Management</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/pending-drivers"><i class="fas fa-user-clock"></i>Pending Drivers</a></li>
                                <li class="list-group-item"><a href="/approved-drivers"><i class="fas fa-user-check"></i>Approved Drivers</a></li>
                                <li class="list-group-item"><a href="/subscription"><i class="fas fa-money-check"></i>Subscription</a></li>
                                <li class="list-group-item"><a href="/drivers-rating"><i class="fas fa-star"></i>Drivers Ratings</a></li>
                            </ul>
                        </li>
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-user-shield"></i>Admin Management</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/admins"><i class="fas fa-user-cog"></i>Admins</a></li>
                            </ul>
                        </li> -->
                        <!-- <li class="list-group-item"><a href="/referral-settings"><i class="fas fa-user-plus"></i>Referral Settings</a></li> -->
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-headset"></i>Support Management</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/title"><i class="fas fa-heading"></i>Ticket Title</a></li>
                                <li class="list-group-item"><a href="/support-tickets"><i class="fas fa-ticket-alt"></i>Support Tickets</a></li>
                            </ul>
                        </li> -->
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-chart-bar"></i>Report</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/report/user-report"><i class="fas fa-user"></i>User Report</a></li>
                                <li class="list-group-item"><a href="/report/driver-report"><i class="fas fa-user-tie"></i>Driver Report</a></li>
                            </ul>
                        </li> -->
                    </ul>
                </div>
            </div>

            <!-- Settings Card -->
            <div class="col">
                <div class="card h-100">
                    <div class="card-header"><i class="fas fa-cog me-2"></i>Settings</div>
                    <ul class="list-group list-group-flush">
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-briefcase"></i>Business Settings</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/general-settings"><i class="fas fa-sliders-h"></i>General Settings</a></li>
                                <li class="list-group-item"><a href="/customization-settings"><i class="fas fa-paint-brush"></i>Customization Settings</a></li>
                            </ul>
                        </li>
                        <li class="list-group-item">
                            <a href="#"><i class="fas fa-mobile-alt"></i>App Settings</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/wallet-settings"><i class="fas fa-wallet"></i>Wallet Settings</a></li>
                                <li class="list-group-item"><a href="/tip-settings"><i class="fas fa-coins"></i>Tip Settings</a></li>
                            </ul>
                        </li> -->
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-plug"></i>Third Party Settings</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/payment-gateway"><i class="fas fa-credit-card"></i>Payment Gateway Settings</a></li>
                                <li class="list-group-item"><a href="/firebase"><i class="fas fa-fire"></i>Firebase Settings</a></li>
                            </ul>
                        </li> -->
                        <!-- <li class="list-group-item">
                            <a href="#"><i class="fas fa-globe"></i>CMS Landing Website</a>
                            <ul class="nested-list">
                                <li class="list-group-item"><a href="/landing-header"><i class="fas fa-heading"></i>Header-Footer</a></li>
                                <li class="list-group-item"><a href="/landing-home"><i class="fas fa-home"></i>Home</a></li>
                            </ul>
                        </li> -->
                    </ul>
                </div>
            </div>
        </div>
    </div>
        <BRow>
            <BCol xl="3" md="6">
              <BCard no-body class="card-animate">
                <BCardBody>
                  <div class="d-flex align-items-center">
                    <div class="flex-grow-1 overflow-hidden">
                      <p class="text-uppercase fw-medium text-muted text-truncate mb-0">
                        {{ $t("drivers_registered") }}
                      </p>
                    </div>
                    <div class="flex-shrink-0">
                      <h5 class="text-success fs-14 mb-0">
                        <i class="ri-arrow-right-up-line fs-13 align-middle"></i>
                        100%
                      </h5>
                    </div>
                  </div>
                  <div class="d-flex align-items-end justify-content-between mt-4">
                    <div>
                      <h4 class="fs-22 fw-semibold ff-secondary mb-4">
                        {{ totalDrivers.total }} 
                      </h4><br>
                      <!-- <Link href="/approved-drivers" class="text-decoration-underline">{{ $t("view_all") }}</Link> -->
                    </div>
                    <div class="avatar-sm flex-shrink-0">
                      <span class="avatar-title bg-success-subtle rounded fs-3">
                        <i class="bx bx-user-circle text-success"></i>
                      </span>
                    </div>
                  </div>
                </BCardBody>
              </BCard>
            </BCol>

            <BCol xl="3" md="6">
              <BCard no-body class="card-animate">
                <BCardBody>
                  <div class="d-flex align-items-center">
                    <div class="flex-grow-1 overflow-hidden">
                      <p class="text-uppercase fw-medium text-muted text-truncate mb-0">
                        {{ $t("approved_drivers") }}
                      </p>
                    </div>
                    <div class="flex-shrink-0">
                      <h5 class="text-success fs-14 mb-0">
                        <i class="ri-arrow-right-up-line fs-13 align-middle"></i>
                         {{ totalDrivers.approve_percentage }} %
                      </h5>
                    </div>  
                  </div>
                  <div class="d-flex align-items-end justify-content-between mt-4">
                    <div>
                      <h4 class="fs-22 fw-semibold ff-secondary mb-4">
                        {{ totalDrivers.approved }} 
                      </h4>
                      <Link href="/approved-drivers" class="text-decoration-underline" v-if="permissions.includes('view-approved-drivers')">{{ $t("view_all") }}</Link>
                    </div>
                    <div class="avatar-sm flex-shrink-0">
                      <span class="avatar-title bg-info-subtle rounded fs-3">
                        <i class="bx bx-user-circle text-info"></i>
                      </span>
                    </div>
                  </div>
                </BCardBody>
              </BCard>
            </BCol>

            <BCol xl="3" md="6">
              <BCard no-body class="card-animate">
                <BCardBody>
                  <div class="d-flex align-items-center">
                    <div class="flex-grow-1 overflow-hidden">
                      <p class="text-uppercase fw-medium text-muted text-truncate mb-0">
                        {{ $t("drivers_approval_waiting") }}
                      </p>
                    </div>
                    <div class="flex-shrink-0">
                      <h5 class="text-danger fs-14 mb-0">
                        <i class="ri-arrow-right-down-line fs-13 align-middle"></i>
                         {{ totalDrivers.decline_percentage }} %
                      </h5>
                    </div>
                  </div>
                  <div class="d-flex align-items-end justify-content-between mt-4">
                    <div>
                      <h4 class="fs-22 fw-semibold ff-secondary mb-4">
                         {{ totalDrivers.declined }}
                      </h4>
                      <Link href="/pending-drivers" class="text-decoration-underline" v-if="permissions.includes('view-approval-pending-drivers')">{{ $t("view_all") }}</Link>
                    </div>
                    <div class="avatar-sm flex-shrink-0">
                      <span class="avatar-title bg-warning-subtle rounded fs-3">
                        <i class="bx bx-user-circle text-warning"></i>
                      </span>
                    </div>
                  </div>
                </BCardBody>
              </BCard>
            </BCol>

            <BCol xl="3" md="6">
              <BCard no-body class="card-animate">
                <BCardBody>
                  <div class="d-flex align-items-center">
                    <div class="flex-grow-1 overflow-hidden">
                      <p class="text-uppercase fw-medium text-muted text-truncate mb-0">
                        {{ $t("users_registered") }}
                      </p>
                    </div>
                    <div class="flex-shrink-0">
                    </div>
                  </div>
                  <div class="d-flex align-items-end justify-content-between mt-4">
                    <div>
                      <h4 class="fs-22 fw-semibold ff-secondary mb-4">
                         {{ totalUsers }} 
                      </h4>
                      <Link href="/users" class="text-decoration-underline" v-if="permissions.includes('view-users')">{{ $t("view_all") }}</Link>
                    </div>
                    <div class="avatar-sm flex-shrink-0">
                      <span class="avatar-title bg-primary-subtle rounded fs-3">
                        <i class="bx bx-user-circle text-primary"></i>
                      </span>
                    </div>
                  </div>
                </BCardBody>
              </BCard>
            </BCol>
          </BRow>

<!-- Notified sos -->
          <BRow>
            <BCard no-body>
              <div class="card-header align-items-center d-flex">
                  <h4 class="card-title mb-0 flex-grow-1">{{ $t("notified_sos") }}</h4>
              </div>
              <BCardBody>
                <BCol v-if="sosRequests.length>0">
                  <div class="table-responsive">
                      <table class="table align-middle position-relative table-nowrap">
                          <thead class="table-active">
                              <tr>
                                  <th scope="col"> {{$t("date")}}</th>
                                  <th scope="col"> {{$t("user_name")}}</th>
                                  <th scope="col"> {{$t("driver_name")}}</th>
                                  <th scope="col"> {{$t("user_type")}}</th>
                                  <th scope="col"> {{$t("action")}}</th>
                              </tr>
                          </thead>
                          <tbody v-if="sosRequests.length > 0">
                              <tr v-for="(result, index) in sosRequests" :key="index">
                                  <td>{{ result.date}}</td> 
                                  <td>{{ result.userName}}</td> 
                                  <td>{{ result.driverName }}</td>
                                  <td>
                                      <BBadge class="text-uppercase">{{ result.is_user ? $t('user') : $t('driver') }} </BBadge>
                                  </td>
                                  <td>
                                      <div class="dropdown">
                                          <a class="text-reset" href="#" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                                              <span class="text-muted fs-18"><i class="mdi mdi-dots-vertical"></i></span>
                                          </a>
                                          <div class="dropdown-menu dropdown-menu-end">
                                              <Link class="dropdown-item" type="button" :href="`rides-request/view/${result.request_id}`" >
                                                  <i class=" bx bx-show-alt align-center text-muted me-2"></i>  {{$t("view")}}
                                              </Link>
                                          </div>
                                      </div>
                                  </td>
                                </tr>
                          </tbody>
                          <tbody v-else>
                              <tr>
                                  <td colspan="10" class="text-center">
                                      <img src="@assets/images/search-file.gif" alt="Loading..." style="width:100px" />
                                      <h5> {{$t("no_data_found")}}</h5>
                                  </td>
                              </tr>
                          </tbody>
                      </table>
                  </div>

                </BCol>
                <BCol xl="12" v-else>
                    <div class="mt-auto text-center">
                        <img src="@assets/images/search-file.gif" width="120"alt="no-data" class="img-fluid">
                        <h5>{{ $t("no_data_found") }}</h5>
                    </div>
                </BCol>
              </BCardBody>
            </BCard>
          </BRow>

  <!-- rides -->
  <BRow>
    <BCol xl="6" md="6">
          <BCard no-body class="card-height-100">
    <BCardHeader class="align-items-center d-flex py-0">
      <BCardTitle class="mb-0 flex-grow-1 p-3">{{ $t("today_trips") }}</BCardTitle>
    </BCardHeader>
    <BCardBody>
      <apexchart class="apex-charts" dir="ltr" height="219" :series="series" :options="chartOptions"></apexchart>

      <div class="table-responsive mt-3">
        <table class="table table-borderless table-sm table-centered align-middle table-nowrap mb-0">
          <tbody class="border-0">
            <tr>
              <td>
                <h4 class="text-truncate fs-14 fs-medium mb-0">
                  <i class="ri-stop-fill align-middle fs-18 text-primary me-2"></i>{{ $t("completed_rides") }}
                </h4>
              </td>
            </tr>
            <tr>
              <td>
                <h4 class="text-truncate fs-14 fs-medium mb-0">
                  <i class="ri-stop-fill align-middle fs-18 text-warning me-2"></i>{{ $t("cancelled_rides") }}
                </h4>
              </td>
            </tr>
            <tr>
              <td>
                <h4 class="text-truncate fs-14 fs-medium mb-0">
                  <i class="ri-stop-fill align-middle fs-18 text-info me-2"></i>{{ $t("scheduled_rides") }}
                </h4>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </BCardBody>
  </BCard>
</BCol>
 
<BCol xl="6" md="6">
<div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("today_earnings") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ todayEarnings.total.toFixed(2) }} </h2>
                            </div>
                      <div>
                      <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-info-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-info icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("by_cash") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ todayEarnings.cash.toFixed(2) }} </h2>
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-success-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-success icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
  </div>
  <div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("by_wallet") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                              <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ todayEarnings.wallet.toFixed(2) }} </h2>
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-warning-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-warning icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("by_card") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                              <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ todayEarnings.card.toFixed(2) }} </h2>                     
                           </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-danger-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-danger icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
  </div>
  <div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("admin_commission") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                              <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ todayEarnings.admin_commision.toFixed(2) }} </h2>                     
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-primary-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-primary icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("drivers_earnings") }}</p>
                         <h2 class="mt-4 ff-secondary fw-semibold">
                              <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ todayEarnings.driver_commision.toFixed(2) }} </h2>                                           </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-dark-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-dark icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
  </div>
</Bcol>

<!-- Overall Trips Chart -->
<BCol xl="12" md="12">
    <BCard no-body class="card-height-100">
      <BCardHeader class="align-items-center d-flex py-0">
        <BCardTitle class="mb-0 flex-grow-1 p-3">{{ $t("overall_trips") }}</BCardTitle>
      </BCardHeader>
      <BCardBody>
        <apexchart class="apex-charts" dir="ltr" height="219" :series="seriesOverallTrip" :options="chartOptionsOverallTrip"></apexchart>
        <div class="table-responsive mt-3">
          <table class="table table-borderless table-sm table-centered align-middle table-nowrap mb-0">
            <tbody class="border-0">
              <tr>
                <td>
                  <h4 class="text-truncate fs-14 fs-medium mb-0">
                    <i class="ri-stop-fill align-middle fs-18 text-secondary me-2"></i>{{ $t("completed_rides") }}
                  </h4>
                </td>
              </tr>
              <tr>
                <td>
                  <h4 class="text-truncate fs-14 fs-medium mb-0">
                    <i class="ri-stop-fill align-middle fs-18 text-danger me-2"></i>{{ $t("cancelled_rides") }}
                  </h4>
                </td>
              </tr>
              <tr>
                <td>
                  <h4 class="text-truncate fs-14 fs-medium mb-0">
                    <i class="ri-stop-fill align-middle fs-18 text-success me-2"></i>{{ $t("scheduled_rides") }}
                  </h4>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </BCardBody>
    </BCard>
  </BCol>
</BRow>


<!-- total earnings -->

<BRow>
  <BCol xl="6" md="6">
    <BCard no-body>
    <BCardBody class="p-0">
      <BRow class="g-0">
        <BCol xxl="12">
          <div class="">
            <BCardHeader class="align-items-center d-flex">
              <BCardTitle class="mb-0 flex-grow-1">{{ $t("overall_earnings") }}</BCardTitle>
            </BCardHeader>
            <apexchart class="apex-charts" height="350" dir="ltr" :series="overall" :options="overallChartOptions"></apexchart>
          </div>
        </BCol>       
      </BRow>
    </BCardBody>
  </BCard>
  </BCol>
  <BCol xl="6" md="6">
<div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("overall_earnings") }}</p>
                         <h2 class="mt-4 ff-secondary fw-semibold">
                              <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ overallEarnings.total.toFixed(2) }} </h2>  
                            </div>
                      <div>
                      <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-danger-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-danger icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("by_cash") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ overallEarnings.cash.toFixed(2) }} </h2>                     
                           </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-warning-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-warning icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
  </div>
  <div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("by_wallet") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ overallEarnings.wallet.toFixed(2) }} </h2>                     
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-success-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-success icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("by_card") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ overallEarnings.card.toFixed(2) }} </h2>                     
                          </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-info-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-info icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
  </div>
  <div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("admin_commission") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ overallEarnings.admin_commision.toFixed(2) }} </h2>                     
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-dark-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-dark icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("drivers_earnings") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                          <span class="counter-value" data-target="97.66">{{currencySymbol}}</span>
                             {{ overallEarnings.driver_commision.toFixed(2) }} </h2>                     
                          </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-primary-subtle rounded-circle fs-2">
                          <i class="bx bx-money text-primary icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
  </div>
</Bcol>
</BRow>


<!-- Cancellation Chart -->
<BRow>
  <BCol xl="6" md="6">
    <BCard no-body>
    <BCardBody class="p-0">
      <BRow class="g-0">
        <BCol xxl="12">
          <div class="">
            <BCardHeader class="align-items-center d-flex">
              <BCardTitle class="mb-0 flex-grow-1">{{ $t("cancellation_chart") }}</BCardTitle>
            </BCardHeader>
            <apexchart class="apex-charts" height="350" dir="ltr" :series="cancellation" :options="cancelChartOptions"></apexchart>
          </div>
        </BCol>       
      </BRow>
    </BCardBody>
  </BCard>
  </BCol>
  <BCol xl="6" md="6">
<div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("total_request_cancelled") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                                {{ cancelledtrips.total_cancelled }}
                          </h2>
                      </div>
                      <div>
                      <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-success-subtle rounded-circle fs-2">
                          <i class="bx bx-user text-success icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
<!-- 
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("cancelled_due_to_no_drivers") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                                {{ cancelledtrips.auto_cancelled }}
                          </h2>
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-danger-subtle rounded-circle fs-2">
                          <i class="bx bx-user text-danger icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div>
          </div>
      </div> -->
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("cancelled_by_users") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                                {{ cancelledtrips.user_cancelled }}
                          </h2>
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-warning-subtle rounded-circle fs-2">
                          <i class="bx bx-user text-warning icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->

    </div>
  <div class="row">
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("cancelled_by_drivers") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                                {{ cancelledtrips.driver_cancelled }}
                          </h2>
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-info-subtle rounded-circle fs-2">
                          <i class="bx bx-user text-info icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->
      <div class="col-md-6">
          <div class="card card-animate">
              <div class="card-body">
                  <div class="d-flex justify-content-between">
                      <div>
                          <p class="fw-medium text-muted mb-0">{{ $t("dispatcher_cancelled") }}</p>
                          <h2 class="mt-4 ff-secondary fw-semibold">
                                {{ cancelledtrips.dispatcher_cancelled }}
                          </h2>
                      </div>
                      <div>
                        <div class="avatar-sm flex-shrink-0">
                        <span class="avatar-title bg-info-subtle rounded-circle fs-2">
                          <i class="bx bx-user text-info icon-lg"></i>
                        </span>
                      </div>
                      </div>
                  </div>
              </div><!-- end card body -->
          </div> <!-- end card-->
      </div> <!-- end col-->


  </div>
</Bcol>
</BRow>
  </Layout>
</template>
<style>
        body {
            background-color: #f4f7fc;
        }
        .dashboard-sections {
            padding: 2rem;
        }
        .card {
            border: none;
            border-radius: 15px;
            transition: transform 0.3s, box-shadow 0.3s;
            background: linear-gradient(145deg, #ffffff, #e6e6e6);
            box-shadow: 5px 5px 15px #d1d1d1, -5px -5px 15px #ffffff;
        }
        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
        }
        .card-header {
            background: linear-gradient(to right, #007bff, #00d4ff);
            color: white;
            border-radius: 15px 15px 0 0;
            padding: 1.5rem;
            font-size: 1.25rem;
            font-weight: bold;
        }
        .list-group-item {
            border: none;
            padding: 0.75rem 1.5rem;
            transition: background 0.3s, color 0.3s;
        }
        .list-group-item:hover {
            background-color: #e7f1ff;
            color: #0056b3;
        }
        .list-group-item a {
            text-decoration: none;
            color: #003afc;
            display: flex;
            align-items: center;
        }
        .list-group-item i {
            margin-right: 0.5rem;
        }
        .nested-list {
            margin-left: 1.5rem;
            border-left: 2px solid #007bff;
            padding-left: 1rem;
        }
    </style>