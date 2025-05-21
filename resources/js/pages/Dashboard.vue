<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import { Head, router } from '@inertiajs/vue3';
import Card from '@/components/ui/card/Card.vue';
import CardTitle from '@/components/ui/card/CardTitle.vue';
import Button from '@/components/ui/button/Button.vue';
import Tooltip from '@/components/ui/tooltip/Tooltip.vue';
import TooltipTrigger from '@/components/ui/tooltip/TooltipTrigger.vue';
import TooltipContent from '@/components/ui/tooltip/TooltipContent.vue';
import { Line, Pie } from 'vue-chartjs';
import {
  Chart as ChartJS,
  Title,
  Tooltip as ChartTooltip,
  Legend,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
  ArcElement
} from 'chart.js';
import { LayoutGrid } from 'lucide-vue-next';

ChartJS.register(Title, ChartTooltip, Legend, LineElement, PointElement, CategoryScale, LinearScale, ArcElement);

const totalEmployees = 52;
const attendanceSummary = {
  present: 44,
  late: 3,
  absent: 2,
  onLeave: 3,
};
const pendingLeaveRequests = 4;
const today = new Date();
const events = [
  { title: 'Weekly Team Meeting', date: '2025-05-20', time: '09:00', description: 'Discuss project updates.' },
  { title: 'Client Presentation', date: '2025-05-21', time: '13:00', description: 'Present Q2 progress.' },
  { title: 'HR Policy Review', date: '2025-05-22', time: '15:00', description: 'Review new HR policies.' },
  { title: 'Product Demo', date: '2025-05-25', time: '11:00', description: 'Demo new features.' },
];
const upcomingEvents = events.filter(e => {
  const d = new Date(e.date);
  return d >= today && d <= new Date(today.getFullYear(), today.getMonth(), today.getDate() + 3);
});

const monthlyAttendance = {
  labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
  datasets: [
    {
      label: 'Present',
      data: [40, 42, 45, 43, 44, 46],
      borderColor: '#2563eb',
      backgroundColor: 'rgba(37,99,235,0.1)',
      fill: true,
      tension: 0.4,
    },
    {
      label: 'Absent',
      data: [2, 1, 3, 2, 2, 1],
      borderColor: '#ef4444',
      backgroundColor: 'rgba(239,68,68,0.1)',
      fill: true,
      tension: 0.4,
    },
    {
      label: 'Late',
      data: [3, 2, 2, 3, 3, 2],
      borderColor: '#f59e42',
      backgroundColor: 'rgba(245,158,66,0.1)',
      fill: true,
      tension: 0.4,
    },
    {
      label: 'On Leave',
      data: [2, 3, 2, 2, 3, 2],
      borderColor: '#10b981',
      backgroundColor: 'rgba(16,185,129,0.1)',
      fill: true,
      tension: 0.4,
    },
  ],
};
const attendanceOptions = {
  responsive: true,
  plugins: {
    legend: { display: true, position: 'top' },
    tooltip: { enabled: true },
    title: { display: false },
  },
  scales: {
    y: { beginAtZero: true, title: { display: true, text: 'Count' } },
    x: { title: { display: true, text: 'Month' } },
  },
};
const leavePie = {
  labels: ['Vacation', 'Sick', 'Emergency', 'Maternity', 'Paternity'],
  datasets: [
    {
      label: 'Leave Type',
      data: [12, 5, 2, 1, 1],
      backgroundColor: ['#2563eb', '#ef4444', '#f59e42', '#10b981', '#a78bfa'],
    },
  ],
};
const leavePieOptions = {
  responsive: true,
  plugins: {
    legend: { display: true, position: 'bottom' },
    tooltip: { enabled: true },
    title: { display: false },
  },
};
const notifications = [
  { type: 'leave', message: 'John Doe filed Sick Leave - pending', date: 'May 19, 2025' },
  { type: 'balance', message: 'Jane Doe has 1 day Vacation Leave remaining', date: 'May 18, 2025' },
  { type: 'overdue', message: 'Mark Lee has an overdue leave approval', date: 'May 17, 2025' },
  { type: 'system', message: 'System maintenance scheduled for May 25', date: 'May 17, 2025' },
];

function formatDate(dateStr) {
  const date = new Date(dateStr);
  const monthAbbr = date.toLocaleString('en-US', { month: 'short' });
  const day = date.getDate();
  const year = date.getFullYear();
  return `${monthAbbr} ${day}, ${year}`;
}

</script>

<template>
  <Head title="Dashboard" />
  <AppLayout :breadcrumbs="[{ title: 'Dashboard', href: '/dashboard' }]">
    <div class="flex flex-col gap-6 p-4">
      <!-- Top Row Widgets -->
      <div class="flex items-center justify-between">
        <h1 class="text-2xl font-bold flex items-center gap-2">
          <LayoutGrid class="w-6 h-6" /> Dashboard
        </h1>
      </div>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <Card class="p-4 flex flex-col items-center justify-center">
          <CardTitle>Total Employees</CardTitle>
          <div class="text-3xl font-bold mt-2">{{ totalEmployees }}</div>
        </Card>
        <Card class="p-4">
          <CardTitle>Today's Attendance</CardTitle>
          <div class="grid grid-cols-2 gap-2 mt-2">
            <div class="flex flex-col items-center">
              <span class="text-green-600 font-semibold">Present</span>
              <span class="font-bold">{{ attendanceSummary.present }}</span>
            </div>
            <div class="flex flex-col items-center">
              <span class="text-yellow-600 font-semibold">Late</span>
              <span class="font-bold">{{ attendanceSummary.late }}</span>
            </div>
            <div class="flex flex-col items-center">
              <span class="text-red-600 font-semibold">Absent</span>
              <span class="font-bold">{{ attendanceSummary.absent }}</span>
            </div>
            <div class="flex flex-col items-center">
              <span class="text-blue-600 font-semibold">On Leave</span>
              <span class="font-bold">{{ attendanceSummary.onLeave }}</span>
            </div>
          </div>
        </Card>
        <Card class="p-4 flex flex-col items-center justify-center">
          <CardTitle>Pending Leave Requests</CardTitle>
          <div class="text-3xl font-bold mt-2">{{ pendingLeaveRequests }}</div>
        </Card>
        <Card class="p-4">
          <CardTitle>Upcoming Events (3 days)</CardTitle>
          <ul class="mt-2 space-y-1">
            <li v-for="event in upcomingEvents" :key="event.title" class="text-sm">
              <span class="font-semibold">{{ event.title }}</span> <span class="text-gray-500">({{ formatDate(event.date) }})</span>
            </li>
            <li v-if="upcomingEvents.length === 0" class="text-gray-400">No events</li>
          </ul>
        </Card>
      </div>
      <!-- Charts Row -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
        <Card class="p-4 col-span-2">
          <div class="flex items-center gap-2 mb-2">
            <CardTitle>Monthly Attendance Trend</CardTitle>
            <Tooltip>
              <TooltipTrigger as-child>
                <span class="ml-1 cursor-pointer text-gray-400">&#9432;</span>
              </TooltipTrigger>
              <TooltipContent>Shows attendance breakdown by month.</TooltipContent>
            </Tooltip>
          </div>
          <Line :data="monthlyAttendance" :options="attendanceOptions" style="max-height:320px" />
        </Card>
        <Card class="p-4">
          <div class="flex items-center gap-2 mb-2">
            <CardTitle>Leave Distribution by Type</CardTitle>
            <Tooltip>
              <TooltipTrigger as-child>
                <span class="ml-1 cursor-pointer text-gray-400">&#9432;</span>
              </TooltipTrigger>
              <TooltipContent>Shows the breakdown of leave types taken.</TooltipContent>
            </Tooltip>
          </div>
          <Pie :data="leavePie" :options="leavePieOptions" style="max-height:320px" />
        </Card>
      </div>
      <!-- Notifications Panel -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
        <Card class="p-4 col-span-2">
          <div class="flex items-center justify-between mb-2">
            <CardTitle>Notifications</CardTitle>
            <Button 
              variant="outline" 
              class="text-xs" 
              @click="router.visit('/leaves')"
            >
              Go to Leave Management
            </Button>
          </div>
          <ul class="space-y-2">
            <li v-for="note in notifications" :key="note.message" class="flex items-center gap-2">
              <span v-if="note.type === 'leave'" class="text-blue-600">&#128197;</span>
              <span v-else-if="note.type === 'balance'" class="text-yellow-600">&#9888;</span>
              <span v-else-if="note.type === 'overdue'" class="text-red-600">&#9201;</span>
              <span v-else class="text-gray-400">&#9432;</span>
              <span>{{ note.message }}</span>
              <span class="ml-auto text-xs text-gray-400">{{ note.date }}</span>
            </li>
          </ul>
        </Card>
        <Card class="p-4 flex flex-col gap-2">
          <CardTitle>System Alerts</CardTitle>
          <div class="bg-blue-50 text-blue-700 rounded p-2 text-sm">System maintenance scheduled for May 25, 2025.</div>
          <div class="bg-yellow-50 text-yellow-700 rounded p-2 text-sm">Some users have low leave balances.</div>
        </Card>
      </div>
    </div>
  </AppLayout>
</template>
