//Pages/Admin/Dashboard.vue
<script setup>
import AdminLayout from "@/Layouts/AdminLayout.vue";
import { ref, onMounted, computed } from "vue";
import { usePage, Link } from "@inertiajs/vue3";
import Chart from "chart.js/auto";

const waterChart = ref(null);
const userDistributionChart = ref(null);
const reportStatusChart = ref(null);

const page = usePage();
const totalUsers = page.props.total_users ?? 0;
const totalStaffs = page.props.total_staffs ?? 0;
const totalReports = page.props.total_reports ?? 0;
const totalCustomers = page.props.total_customers ?? 0;
const monthlyConsumption = page.props.monthly_consumption ?? Array(12).fill(0);
const currentMonthConsumption = page.props.current_month_consumption ?? 0;
const averageConsumption = page.props.average_consumption ?? 0;
const peakUsage = page.props.peak_usage ?? 0;
const growthPercentage = page.props.growth_percentage ?? 0;
const resolutionRate = page.props.resolution_rate ?? 0;
const recentActivities = page.props.recent_activities ?? [];
const canViewActivityLog =
    page.props.auth?.user?.can?.("view-activity-log") ?? false;

// Activity display helpers
const activityConfig = {
    created: {
        icon: "plus-circle",
        bg: "bg-blue-50",
        text: "text-blue-500",
        title: "New record created",
    },
    updated: {
        icon: "pencil-square",
        bg: "bg-green-50",
        text: "text-green-500",
        title: "Record updated",
    },
    deleted: {
        icon: "trash",
        bg: "bg-red-50",
        text: "text-red-500",
        title: "Record deleted",
    },
    logged_in: {
        icon: "arrow-right-on-rectangle",
        bg: "bg-purple-50",
        text: "text-purple-500",
        title: "User login",
    },
    logged_out: {
        icon: "arrow-left-on-rectangle",
        bg: "bg-gray-50",
        text: "text-gray-500",
        title: "User logout",
    },
    verification_success: {
        icon: "shield-check",
        bg: "bg-teal-50",
        text: "text-teal-500",
        title: "Verification passed",
    },
    verification_failed: {
        icon: "shield-exclamation",
        bg: "bg-amber-50",
        text: "text-amber-500",
        title: "Verification failed",
    },
    default: {
        icon: "sparkles",
        bg: "bg-indigo-50",
        text: "text-indigo-500",
        title: "System activity",
    },
    report_created: {
        icon: "flag",
        bg: "bg-amber-50",
        text: "text-amber-500",
        title: "New Report Submitted",
    },
    report_status_changed: {
        icon: "arrow-path",
        bg: "bg-blue-50",
        text: "text-blue-500",
        title: "Report Status Updated",
    },
};

const getActivityConfig = (activity) => {
    if (activity.subject_type === "App\\Models\\Report") {
        return activity.event === "report_created"
            ? activityConfig.report_created
            : activityConfig.report_status_changed;
    }
    return activityConfig[activity.event] || activityConfig.default;
};

const getActivityIcon = (activity) => {
    return getActivityConfig(activity).icon;
};

const getActivityBgColor = (activity) => {
    return getActivityConfig(activity).bg;
};

const getActivityTextColor = (activity) => {
    return getActivityConfig(activity).text;
};

const getActivityTitle = (activity) => {
    return getActivityConfig(activity).title;
};

const formatTimeAgo = (dateString) => {
    const date = new Date(dateString);
    const now = new Date();
    const seconds = Math.floor((now - date) / 1000);
    const intervals = {
        year: 31536000,
        month: 2592000,
        week: 604800,
        day: 86400,
        hour: 3600,
        minute: 60,
    };

    if (seconds < 60) return "Just now";

    for (const [unit, secondsInUnit] of Object.entries(intervals)) {
        const interval = Math.floor(seconds / secondsInUnit);
        if (interval >= 1) {
            return `${interval} ${unit}${interval === 1 ? "" : "s"} ago`;
        }
    }

    return date.toLocaleDateString("en-US", {
        month: "short",
        day: "numeric",
        year: date.getFullYear() !== now.getFullYear() ? "numeric" : undefined,
    });
};

// Get current month name
const currentMonthName = computed(() => {
    const months = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
    ];
    return months[new Date().getMonth()];
});

// Get current year
const currentYear = computed(() => {
    return new Date().getFullYear();
});

// Get peak usage month
const peakUsageMonth = computed(() => {
    const months = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
    ];
    const maxConsumption = Math.max(...monthlyConsumption);
    const peakIndex = monthlyConsumption.indexOf(maxConsumption);
    return months[peakIndex];
});

// Chart initialization
onMounted(() => {
    // Water Consumption Bar Chart (Building Style) - Fixed white space
    if (waterChart.value) {
        new Chart(waterChart.value, {
            type: "bar",
            data: {
                labels: [
                    "Jan",
                    "Feb",
                    "Mar",
                    "Apr",
                    "May",
                    "Jun",
                    "Jul",
                    "Aug",
                    "Sep",
                    "Oct",
                    "Nov",
                    "Dec",
                ],
                datasets: [
                    {
                        label: "Water Consumption (m³)",
                        data: monthlyConsumption,
                        backgroundColor: [
                            "rgba(79, 70, 229, 0.8)",
                            "rgba(99, 102, 241, 0.8)",
                            "rgba(129, 140, 248, 0.8)",
                            "rgba(147, 197, 253, 0.8)",
                            "rgba(96, 165, 250, 0.8)",
                            "rgba(59, 130, 246, 0.8)",
                            "rgba(37, 99, 235, 0.8)",
                            "rgba(29, 78, 216, 0.8)",
                            "rgba(30, 64, 175, 0.8)",
                            "rgba(30, 58, 138, 0.8)",
                            "rgba(23, 37, 84, 0.8)",
                            "rgba(12, 74, 110, 0.8)",
                        ],
                        borderColor: [
                            "rgb(79, 70, 229)",
                            "rgb(99, 102, 241)",
                            "rgb(129, 140, 248)",
                            "rgb(147, 197, 253)",
                            "rgb(96, 165, 250)",
                            "rgb(59, 130, 246)",
                            "rgb(37, 99, 235)",
                            "rgb(29, 78, 216)",
                            "rgb(30, 64, 175)",
                            "rgb(30, 58, 138)",
                            "rgb(23, 37, 84)",
                            "rgb(12, 74, 110)",
                        ],
                        borderWidth: 2,
                        borderRadius: 8,
                        borderSkipped: false,
                    },
                ],
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { display: false },
                    tooltip: {
                        backgroundColor: "white",
                        titleColor: "#1f2937",
                        bodyColor: "#4b5563",
                        borderColor: "#e5e7eb",
                        borderWidth: 1,
                        padding: 12,
                        boxShadow: "0 4px 12px rgba(0, 0, 0, 0.08)",
                        callbacks: {
                            label: (context) => `${context.parsed.y} m³`,
                        },
                    },
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        grid: {
                            drawBorder: false,
                            color: "#f3f4f6",
                            drawTicks: false,
                        },
                        ticks: {
                            color: "#9ca3af",
                            callback: (value) => `${value} m³`,
                            padding: 8,
                            maxTicksLimit: 6, // Reduced ticks to minimize space
                        },
                    },
                    x: {
                        grid: {
                            display: false,
                            drawBorder: false,
                        },
                        ticks: {
                            color: "#9ca3af",
                            padding: 8,
                        },
                    },
                },
                layout: {
                    padding: {
                        bottom: 10, // Reduced bottom padding
                        top: 10,
                    },
                },
                interaction: { intersect: false, mode: "index" },
            },
        });
    }

    // User Distribution Doughnut Chart
    if (userDistributionChart.value) {
        new Chart(userDistributionChart.value, {
            type: "doughnut",
            data: {
                labels: ["Customers", "Staff", "Other Users"],
                datasets: [
                    {
                        data: [
                            totalCustomers,
                            totalStaffs,
                            totalUsers - totalCustomers - totalStaffs,
                        ],
                        backgroundColor: [
                            "rgba(139, 92, 246, 0.8)",
                            "rgba(34, 197, 94, 0.8)",
                            "rgba(59, 130, 246, 0.8)",
                        ],
                        borderColor: [
                            "rgb(139, 92, 246)",
                            "rgb(34, 197, 94)",
                            "rgb(59, 130, 246)",
                        ],
                        borderWidth: 2,
                        hoverOffset: 15,
                    },
                ],
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                cutout: "70%",
                plugins: {
                    legend: {
                        position: "bottom",
                        labels: {
                            padding: 20,
                            usePointStyle: true,
                            pointStyle: "circle",
                            color: "#6b7280",
                        },
                    },
                    tooltip: {
                        backgroundColor: "white",
                        titleColor: "#1f2937",
                        bodyColor: "#4b5563",
                        borderColor: "#e5e7eb",
                        borderWidth: 1,
                        padding: 12,
                        boxShadow: "0 4px 12px rgba(0, 0, 0, 0.08)",
                    },
                },
            },
        });
    }

    // Report Status Doughnut Chart
    if (reportStatusChart.value) {
        new Chart(reportStatusChart.value, {
            type: "doughnut",
            data: {
                labels: ["Resolved", "Pending", "In Progress"],
                datasets: [
                    {
                        data: [
                            Math.round(totalReports * (resolutionRate / 100)),
                            Math.round(totalReports * 0.3),
                            Math.round(totalReports * 0.2),
                        ],
                        backgroundColor: [
                            "rgba(34, 197, 94, 0.8)",
                            "rgba(249, 115, 22, 0.8)",
                            "rgba(59, 130, 246, 0.8)",
                        ],
                        borderColor: [
                            "rgb(34, 197, 94)",
                            "rgb(249, 115, 22)",
                            "rgb(59, 130, 246)",
                        ],
                        borderWidth: 2,
                        hoverOffset: 15,
                    },
                ],
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                cutout: "70%",
                plugins: {
                    legend: {
                        position: "bottom",
                        labels: {
                            padding: 20,
                            usePointStyle: true,
                            pointStyle: "circle",
                            color: "#6b7280",
                        },
                    },
                    tooltip: {
                        backgroundColor: "white",
                        titleColor: "#1f2937",
                        bodyColor: "#4b5563",
                        borderColor: "#e5e7eb",
                        borderWidth: 1,
                        padding: 12,
                        boxShadow: "0 4px 12px rgba(0, 0, 0, 0.08)",
                    },
                },
            },
        });
    }
});
</script>

<template>
    <AdminLayout>
        <!-- Header Section -->
        <div class="mb-8">
            <div class="flex items-center justify-between mb-3">
                <div>
                    <h1 class="text-3xl font-bold text-gray-900">
                        Dashboard Overview
                    </h1>
                    <p class="text-gray-500 mt-2">
                        Welcome back! Here's what's happening with your water
                        management system today.
                    </p>
                </div>
                <div class="flex items-center space-x-3">
                    <div
                        class="bg-white rounded-lg px-4 py-2 shadow-sm border border-gray-200"
                    >
                        <span class="text-sm font-medium text-gray-700"
                            >{{ currentMonthName }} {{ currentYear }}</span
                        >
                    </div>
                </div>
            </div>
        </div>

        <!-- Stats Grid - Vertical Layout -->
        <div class="grid grid-cols-1 lg:grid-cols-4 gap-6 mb-8">
            <!-- Total Users Card -->
            <div
                class="bg-gradient-to-br from-blue-500 to-blue-600 rounded-2xl shadow-lg p-6 text-white"
            >
                <div class="flex flex-col h-full">
                    <div class="flex items-center justify-between mb-4">
                        <div
                            class="p-3 rounded-xl bg-white/20 backdrop-blur-sm"
                        >
                            <svg
                                class="w-6 h-6"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z"
                                />
                            </svg>
                        </div>
                        <div class="text-right">
                            <div class="text-sm opacity-90">Total Users</div>
                            <div class="text-2xl font-bold">
                                {{ totalUsers }}
                            </div>
                        </div>
                    </div>
                    <div class="mt-auto text-sm opacity-80">
                        All system users
                    </div>
                </div>
            </div>

            <!-- Active Staff Card -->
            <div
                class="bg-gradient-to-br from-green-500 to-green-600 rounded-2xl shadow-lg p-6 text-white"
            >
                <div class="flex flex-col h-full">
                    <div class="flex items-center justify-between mb-4">
                        <div
                            class="p-3 rounded-xl bg-white/20 backdrop-blur-sm"
                        >
                            <svg
                                class="w-6 h-6"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M10 6H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V8a2 2 0 00-2-2h-5m-4 0V5a2 2 0 114 0v1m-4 0a2 2 0 104 0m-5 8a2 2 0 100-4 2 2 0 000 4zm0 0c1.306 0 2.417.835 2.83 2M9 14a3.001 3.001 0 00-2.83 2"
                                />
                            </svg>
                        </div>
                        <div class="text-right">
                            <div class="text-sm opacity-90">Active Staff</div>
                            <div class="text-2xl font-bold">
                                {{ totalStaffs }}
                            </div>
                        </div>
                    </div>
                    <div class="mt-auto text-sm opacity-80">
                        System staff members
                    </div>
                </div>
            </div>

            <!-- Customers Card -->
            <div
                class="bg-gradient-to-br from-purple-500 to-purple-600 rounded-2xl shadow-lg p-6 text-white"
            >
                <div class="flex flex-col h-full">
                    <div class="flex items-center justify-between mb-4">
                        <div
                            class="p-3 rounded-xl bg-white/20 backdrop-blur-sm"
                        >
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                viewBox="0 0 24 24"
                                fill="currentColor"
                                class="size-6"
                            >
                                <path
                                    fill-rule="evenodd"
                                    d="M8.25 6.75a3.75 3.75 0 1 1 7.5 0 3.75 3.75 0 0 1-7.5 0ZM15.75 9.75a3 3 0 1 1 6 0 3 3 0 0 1-6 0ZM2.25 9.75a3 3 0 1 1 6 0 3 3 0 0 1-6 0ZM6.31 15.117A6.745 6.745 0 0 1 12 12a6.745 6.745 0 0 1 6.709 7.498.75.75 0 0 1-.372.568A12.696 12.696 0 0 1 12 21.75c-2.305 0-4.47-.612-6.337-1.684a.75.75 0 0 1-.372-.568 6.787 6.787 0 0 1 1.019-4.38Z"
                                    clip-rule="evenodd"
                                />
                                <path
                                    d="M5.082 14.254a8.287 8.287 0 0 0-1.308 5.135 9.687 9.687 0 0 1-1.764-.44l-.115-.04a.563.563 0 0 1-.373-.487l-.01-.121a3.75 3.75 0 0 1 3.57-4.047ZM20.226 19.389a8.287 8.287 0 0 0-1.308-5.135 3.75 3.75 0 0 1 3.57 4.047l-.01.121a.563.563 0 0 1-.373.486l-.115.04c-.567.2-1.156.349-1.764.441Z"
                                />
                            </svg>
                        </div>
                        <div class="text-right">
                            <div class="text-sm opacity-90">Customers</div>
                            <div class="text-2xl font-bold">
                                {{ totalCustomers }}
                            </div>
                        </div>
                    </div>
                    <div class="mt-auto text-sm opacity-80">
                        Water service customers
                    </div>
                </div>
            </div>

            <!-- Reports Card -->
            <div
                class="bg-gradient-to-br from-amber-500 to-amber-600 rounded-2xl shadow-lg p-6 text-white"
            >
                <div class="flex flex-col h-full">
                    <div class="flex items-center justify-between mb-4">
                        <div
                            class="p-3 rounded-xl bg-white/20 backdrop-blur-sm"
                        >
                            <svg
                                class="w-6 h-6"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"
                                />
                            </svg>
                        </div>
                        <div class="text-right">
                            <div class="text-sm opacity-90">Reports</div>
                            <div class="text-2xl font-bold">
                                {{ totalReports }}
                            </div>
                        </div>
                    </div>
                    <div class="mt-auto text-sm opacity-80">
                        Water service reports
                    </div>
                </div>
            </div>
        </div>

        <!-- Analytics Section -->
        <div class="grid grid-cols-1 xl:grid-cols-3 gap-6 mb-8">
            <!-- Water Consumption Chart - Fixed white space -->
            <div
                class="xl:col-span-2 bg-white rounded-2xl shadow-sm border border-gray-300  p-6"
            >
                <div
                    class="flex flex-col sm:flex-row sm:items-center sm:justify-between mb-6"
                >
                    <div>
                        <h2 class="text-xl font-bold text-gray-900 mb-2">
                            Water Consumption Analytics
                        </h2>
                        <p class="text-sm text-gray-500">
                            Monthly water consumption trends for
                            {{ currentYear }}
                        </p>
                    </div>
                    <div class="flex items-center mt-2 sm:mt-0 space-x-4">
                        <div class="flex items-center">
                            <div
                                class="w-3 h-3 rounded-full bg-indigo-500 mr-2"
                            ></div>
                            <span class="text-sm text-gray-600"
                                >Consumption (m³)</span
                            >
                        </div>
                    </div>
                </div>
                <!-- Reduced height and removed extra padding -->
                <div class="h-[72vh]">
                    <canvas ref="waterChart"></canvas>
                </div>
            </div>

            <!-- Doughnut Charts Section -->
            <div class="space-y-6">
                <!-- User Distribution Chart -->
                <div
                    class="bg-white rounded-2xl shadow-sm border border-gray-300 p-6"
                >
                    <div class="flex items-center justify-between mb-4">
                        <h3 class="text-lg font-semibold text-gray-900">
                            User Distribution
                        </h3>
                    </div>
                    <div class="h-60">
                        <canvas ref="userDistributionChart"></canvas>
                    </div>
                </div>

                <!-- Report Status Chart -->
                <div
                    class="bg-white rounded-2xl shadow-sm border border-gray-300 p-6"
                >
                    <div class="flex items-center justify-between mb-4">
                        <h3 class="text-lg font-semibold text-gray-900">
                            Report Status
                        </h3>
                    </div>
                    <div class="h-60">
                        <canvas ref="reportStatusChart"></canvas>
                    </div>
                </div>
            </div>
        </div>

        <!-- Recent Activity Section - Modern Card Layout -->
        <div
            class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden"
        >
            <div
                class="p-6 border-b border-gray-100 flex items-center justify-between"
            >
                <h2 class="text-xl font-bold text-gray-900 flex items-center">
                    <svg
                        class="w-5 h-5 text-indigo-500 mr-2"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                    >
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                        />
                    </svg>
                    Recent Activity
                </h2>
                <Link
                    v-if="canViewActivityLog"
                    :href="route('admin.activity-logs')"
                    class="text-sm font-medium text-indigo-600 hover:text-indigo-800 transition-colors flex items-center gap-1 bg-indigo-50 px-4 py-2 rounded-lg hover:bg-indigo-100"
                >
                    View All
                    <svg
                        class="w-4 h-4"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                    >
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M9 5l7 7-7 7"
                        />
                    </svg>
                </Link>
            </div>

            <!-- Modern Activity Cards Layout -->
            <div class="p-6">
                <div class="space-y-4">
                    <div
                        v-for="activity in recentActivities"
                        :key="activity.id"
                        class="flex items-start p-4 rounded-xl border border-gray-100 hover:border-gray-200 hover:shadow-sm transition-all duration-200 group"
                    >
                        <!-- Activity Icon -->
                        <div
                            :class="`p-3 rounded-xl ${getActivityBgColor(
                                activity
                            )} mr-4 group-hover:scale-105 transition-transform duration-200`"
                        >
                            <svg
                                v-if="
                                    getActivityIcon(activity) === 'plus-circle'
                                "
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 01118 0z"
                                />
                            </svg>
                            <svg
                                v-else-if="
                                    getActivityIcon(activity) ===
                                    'pencil-square'
                                "
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
                                />
                            </svg>
                            <svg
                                v-else-if="
                                    getActivityIcon(activity) === 'trash'
                                "
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5-4h4m-4 0H6a1 1 0 00-1 1v1m14 0V4a1 1 0 00-1-1h-4"
                                />
                            </svg>
                            <svg
                                v-else-if="
                                    getActivityIcon(activity) ===
                                    'arrow-right-on-rectangle'
                                "
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M11 16l-4-4m0 0l4-4m-4 4h14m-5 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h7a3 3 0 013 3v1"
                                />
                            </svg>
                            <svg
                                v-else-if="
                                    getActivityIcon(activity) === 'shield-check'
                                "
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"
                                />
                            </svg>
                            <svg
                                v-else-if="getActivityIcon(activity) === 'flag'"
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"
                                />
                            </svg>
                            <svg
                                v-else-if="
                                    getActivityIcon(activity) === 'arrow-path'
                                "
                                class="w-5 h-5"
                                :class="getActivityTextColor(activity)"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0013.803-3.7M4.031 9.865a8.25 8.25 0 0113.803-3.7l3.181 3.182m0-4.991v4.99"
                                />
                            </svg>
                        </div>

                        <!-- Activity Content -->
                        <div class="flex-1 min-w-0">
                            <div class="flex items-start justify-between mb-2">
                                <h3
                                    class="text-sm font-semibold text-gray-900 truncate"
                                >
                                    {{ getActivityTitle(activity) }}
                                </h3>
                                <span
                                    class="text-xs text-gray-400 ml-2 whitespace-nowrap"
                                >
                                    {{ formatTimeAgo(activity.created_at) }}
                                </span>
                            </div>
                            <p class="text-sm text-gray-600 mb-1">
                                {{ activity.description }}
                                <span
                                    v-if="activity.properties?.tracking_code"
                                    class="font-medium text-gray-700"
                                >
                                    ({{ activity.properties.tracking_code }})
                                </span>
                            </p>
                            <div
                                class="flex items-center text-xs text-gray-500"
                            >
                                <span
                                    >By
                                    {{ activity.causer_name || "System" }}</span
                                >
                                <span
                                    v-if="activity.properties?.old_status"
                                    class="ml-3 px-2 py-1 rounded-full bg-gray-100 text-gray-600"
                                >
                                    {{ activity.properties.old_status }} →
                                    {{ activity.properties.new_status }}
                                </span>
                            </div>
                        </div>
                    </div>

                    <!-- Empty State -->
                    <div
                        v-if="recentActivities.length === 0"
                        class="text-center py-12"
                    >
                        <div
                            class="w-16 h-16 mx-auto mb-4 rounded-full bg-gray-100 flex items-center justify-center"
                        >
                            <svg
                                class="w-8 h-8 text-gray-400"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 24 24"
                            >
                                <path
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                    stroke-width="2"
                                    d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                                />
                            </svg>
                        </div>
                        <p class="text-gray-500 text-lg font-medium mb-2">
                            No recent activities
                        </p>
                        <p class="text-gray-400 text-sm">
                            Activities will appear here as they happen
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </AdminLayout>
</template>
