<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import { Head } from '@inertiajs/vue3';
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { Calendar, Plus, Search, MoreVertical, Pencil, Trash2 } from 'lucide-vue-next';
import Card from '@/components/ui/card/Card.vue';
import Dialog from '@/components/ui/dialog/Dialog.vue';
import DialogTrigger from '@/components/ui/dialog/DialogTrigger.vue';
import DialogContent from '@/components/ui/dialog/DialogContent.vue';
import DialogHeader from '@/components/ui/dialog/DialogHeader.vue';
import DialogTitle from '@/components/ui/dialog/DialogTitle.vue';
import DialogDescription from '@/components/ui/dialog/DialogDescription.vue';
import DialogFooter from '@/components/ui/dialog/DialogFooter.vue';
import Button from '@/components/ui/button/Button.vue';

interface Event {
    date: string;
    time?: string;
    timeEnd?: string;
    title: string;
    description: string;
}

const breadcrumbs = [
    { title: 'Calendar', href: '/calendar' },
];

const events = ref<Event[]>([
    {
        date: '2025-05-02', time: '09:00', timeEnd: '10:00', title: 'Weekly Team Meeting', description: 'Discuss project updates and blockers with the whole team.'
    },
    {
        date: '2025-05-03', time: '13:00', timeEnd: '14:00', title: 'Client Presentation', description: 'Present Q2 progress to the client and answer questions.'
    },
    {
        date: '2025-05-05', time: '15:00', timeEnd: '16:00', title: 'HR Policy Review', description: 'Review new HR policies and compliance requirements.'
    },
    {
        date: '2025-05-07', time: '11:00', timeEnd: '12:00', title: 'Product Demo', description: 'Demo the new product features to the sales team.'
    },
    {
        date: '2025-05-10', time: '17:00', title: 'Project Deadline', description: 'Final deadline for submitting Q2 deliverables.'
    },
    {
        date: '2025-05-12', time: '10:00', timeEnd: '11:30', title: 'One-on-One', description: 'Monthly one-on-one meeting with your manager.'
    },
    {
        date: '2025-05-15', time: '14:00', timeEnd: '15:00', title: 'Department All-Hands', description: 'All-hands meeting for department-wide announcements.'
    },
    {
        date: '2025-05-18', time: '12:00', timeEnd: '13:00', title: 'Lunch & Learn', description: 'Team lunch with a guest speaker on productivity.'
    },
    {
        date: '2025-05-20', time: '16:00', timeEnd: '17:00', title: 'Sprint Planning', description: 'Plan tasks and priorities for the next sprint.'
    },
    {
        date: '2025-06-01', time: '09:30', timeEnd: '10:30', title: 'Monthly Kickoff', description: 'Kickoff meeting for June projects and goals.'
    },
    {
        date: '2025-06-05', time: '15:00', timeEnd: '16:00', title: 'Performance Review', description: 'Quarterly performance review with HR.'
    },
    {
        date: '2025-06-10', time: '13:00', timeEnd: '14:00', title: 'Team Building Activity', description: 'Outdoor team building and games.'
    },
    {
        date: '2025-06-15', time: '11:00', timeEnd: '12:00', title: 'Budget Planning', description: 'Plan the budget for the next quarter.'
    },
    {
        date: '2025-06-20', time: '10:00', timeEnd: '11:00', title: 'Customer Feedback Session', description: 'Gather feedback from key customers.'
    }
]);

const today = new Date();
const currentMonth = ref(today.getMonth());
const currentYear = ref(today.getFullYear());

const monthNames = [
    'January', 'February', 'March', 'April', 'May', 'June',
    'July', 'August', 'September', 'October', 'November', 'December'
];

const daysInMonth = computed(() => {
    return new Date(currentYear.value, currentMonth.value + 1, 0).getDate();
});

const firstDayOfWeek = computed(() => {
    return new Date(currentYear.value, currentMonth.value, 1).getDay();
});

const filterRange = ref(7);

const filteredEvents = computed(() => {
    const now = new Date();
    const end = new Date(now);
    end.setDate(now.getDate() + filterRange.value);
    return events.value.filter(event => {
        const eventDate = new Date(event.date + 'T' + (event.time || '00:00'));
        return eventDate >= now && eventDate <= end;
    }).sort((a, b) => {
        const aDate = new Date(a.date + 'T' + (a.time || '00:00'));
        const bDate = new Date(b.date + 'T' + (b.time || '00:00'));
        return aDate.getTime() - bDate.getTime();
    });
});

function prevMonth() {
    if (currentMonth.value === 0) {
        currentMonth.value = 11;
        currentYear.value--;
    } else {
        currentMonth.value--;
    }
}

function nextMonth() {
    if (currentMonth.value === 11) {
        currentMonth.value = 0;
        currentYear.value++;
    } else {
        currentMonth.value++;
    }
}

function getEventsForDay(day: number) {
    const month = String(currentMonth.value + 1).padStart(2, '0');
    const dateStr = `${currentYear.value}-${month}-${String(day).padStart(2, '0')}`;
    return events.value.filter(e => e.date === dateStr);
}

function isToday(day: number) {
    const d = new Date();
    return (
        d.getDate() === day &&
        d.getMonth() === currentMonth.value &&
        d.getFullYear() === currentYear.value
    );
}

const allYears = computed(() => {
    const years = events.value.map(e => Number(e.date.split('-')[0]));
    return Array.from(new Set(years)).sort((a, b) => a - b);
});

const allMonths = [
    'January', 'February', 'March', 'April', 'May', 'June',
    'July', 'August', 'September', 'October', 'November', 'December'
];

const selectedYear = ref(today.getFullYear());
const selectedMonth = ref<number | null>(null);
const selectedDay = ref('');

const allEventsSearch = ref('');

const allEventsFiltered = computed(() => {
    return events.value.filter(event => {
        const [year, month, day] = event.date.split('-').map(Number);
        if (selectedYear.value && year !== selectedYear.value) return false;
        if (selectedMonth.value !== null && month !== selectedMonth.value + 1) return false;
        if (selectedDay.value) {
            const selected = new Date(selectedDay.value);
            return year === selected.getFullYear() && month === selected.getMonth() + 1 && day === selected.getDate();
        }
        if (allEventsSearch.value) {
            const searchLower = allEventsSearch.value.toLowerCase();
            if (!event.title.toLowerCase().includes(searchLower) && 
                !event.description.toLowerCase().includes(searchLower)) {
                return false;
            }
        }
        return true;
    }).sort((a, b) => {
        const aDate = new Date(a.date + 'T' + (a.time || '00:00'));
        const bDate = new Date(b.date + 'T' + (b.time || '00:00'));
        return aDate.getTime() - bDate.getTime();
    });
});

const allEventsCurrentPage = ref(1);
const allEventsPerPage = 5;

const paginatedAllEvents = computed(() => {
    const start = (allEventsCurrentPage.value - 1) * allEventsPerPage;
    const end = start + allEventsPerPage;
    return allEventsFiltered.value.slice(start, end);
});

const allEventsTotalPages = computed(() => Math.ceil(allEventsFiltered.value.length / allEventsPerPage));

function changeAllEventsPage(page: number) {
    allEventsCurrentPage.value = page;
}

const newEvent = ref({
    title: '',
    description: '',
    date: '',
    timeStart: '',
    timeEnd: '',
});

const showEventModal = ref(false);
const selectedEvent = ref<Event | null>(null);
const showDeleteModal = ref(false);
const eventToDelete = ref<Event | null>(null);
const isEditing = ref(false);

const showKebabMenu = ref<number | null>(null);

function openEventModal(event: Event) {
    selectedEvent.value = event;
    showEventModal.value = true;
}

function closeEventModal() {
    showEventModal.value = false;
    selectedEvent.value = null;
    isEditing.value = false;
}

function openDeleteModal(event: Event) {
    eventToDelete.value = event;
    showDeleteModal.value = true;
}

function closeDeleteModal() {
    showDeleteModal.value = false;
    eventToDelete.value = null;
}

function deleteEvent() {
    if (eventToDelete.value) {
        events.value = events.value.filter(e => e !== eventToDelete.value);
        closeDeleteModal();
    }
}

function startEditing(event: Event) {
    selectedEvent.value = {
        title: event.title,
        description: event.description,
        date: event.date,
        time: event.time,
        timeEnd: event.timeEnd
    };
    isEditing.value = true;
    showEventModal.value = true;
}

function saveEvent() {
    if (selectedEvent.value) {
        const originalEvent = events.value.find(e => 
            e.date === selectedEvent.value?.date && 
            e.time === selectedEvent.value?.time && 
            e.title === selectedEvent.value?.title
        );

        if (originalEvent) {
            originalEvent.title = selectedEvent.value.title;
            originalEvent.description = selectedEvent.value.description;
            originalEvent.date = selectedEvent.value.date;
            originalEvent.time = selectedEvent.value.time;
            originalEvent.timeEnd = selectedEvent.value.timeEnd;
        }
        closeEventModal();
    }
}

function formatTime(time: string) {
    if (!time) return '';
    const [hour, minute] = time.split(':').map(Number);
    const ampm = hour >= 12 ? 'PM' : 'AM';
    const hour12 = hour % 12 || 12;
    return `${hour12}:${minute.toString().padStart(2, '0')} ${ampm}`;
}

function formatDate(dateStr: string) {
    const date = new Date(dateStr);
    const monthAbbr = date.toLocaleString('en-US', { month: 'short' });
    const day = date.getDate();
    const year = date.getFullYear();
    return `${monthAbbr} ${day}, ${year}`;
}

function formatEventDateTime(event: { date: string; time?: string; timeEnd?: string }) {
    const date = formatDate(event.date);
    let time = '';
    if (event.time) {
        time = formatTime(event.time);
        if (event.timeEnd) {
            time += ' - ' + formatTime(event.timeEnd);
        }
        return `${date} | ${time}`;
    }
    return date;
}

function truncateDescription(description: string, maxLength: number = 100) {
    return description.length > maxLength ? description.slice(0, maxLength) + '...' : description;
}

function toggleKebabMenu(index: number) {
    showKebabMenu.value = showKebabMenu.value === index ? null : index;
}

function closeKebabMenu() {
    showKebabMenu.value = null;
}

// Add click outside handler
onMounted(() => {
    document.addEventListener('click', (e) => {
        if (!(e.target as HTMLElement).closest('.kebab-menu')) {
            closeKebabMenu();
        }
    });
});

onUnmounted(() => {
    document.removeEventListener('click', closeKebabMenu);
});
</script>

<template>
    <Head title="Calendar" />
    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="flex flex-col gap-6 p-6">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl font-bold flex items-center gap-2">
                    <Calendar class="w-6 h-6" /> Calendar
                </h1>
                <Dialog>
                    <DialogTrigger as-child>
                        <button class="flex items-center gap-2 px-4 py-2 bg-primary text-white rounded-lg shadow hover:bg-primary/90">
                            <Plus class="w-4 h-4" /> Create Event
                        </button>
                    </DialogTrigger>
                    <DialogContent>
                        <DialogHeader>
                            <DialogTitle>Create Event</DialogTitle>
                            <DialogDescription>Fill in the details below to add a new event to the calendar.</DialogDescription>
                        </DialogHeader>
                        <form class="flex flex-col gap-4 mt-4">
                            <div>
                                <label class="block text-sm font-medium mb-1">Event Title</label>
                                <input v-model="newEvent.title" type="text" class="w-full border rounded px-3 py-2" placeholder="Enter event title" />
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-1">Description</label>
                                <textarea v-model="newEvent.description" class="w-full border rounded px-3 py-2" placeholder="Enter event description"></textarea>
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-1">Date</label>
                                <input v-model="newEvent.date" type="date" class="w-full border rounded px-3 py-2" />
                            </div>
                            <div class="flex gap-2">
                                <div class="flex-1">
                                    <label class="block text-sm font-medium mb-1">Time Start</label>
                                    <input v-model="newEvent.timeStart" type="time" class="w-full border rounded px-3 py-2" />
                                </div>
                                <div class="flex-1">
                                    <label class="block text-sm font-medium mb-1">Time End <span class="text-xs text-gray-400">(optional)</span></label>
                                    <input v-model="newEvent.timeEnd" type="time" class="w-full border rounded px-3 py-2" />
                                </div>
                            </div>
                        </form>
                        <DialogFooter class="mt-6">
                            <Button variant="outline" type="button">Cancel</Button>
                            <Button type="button">Create</Button>
                        </DialogFooter>
                    </DialogContent>
                </Dialog>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <!-- Calendar -->
                <Card class="p-4">
                    <div class="flex items-center justify-between mb-4">
                        <button @click="prevMonth" class="text-gray-400 hover:text-gray-600">&lt;</button>
                        <span class="font-semibold">{{ monthNames[currentMonth] }} {{ currentYear }}</span>
                        <button @click="nextMonth" class="text-gray-400 hover:text-gray-600">&gt;</button>
                    </div>
                    <div class="grid grid-cols-7 gap-2 text-center text-sm font-medium text-gray-500 mb-2">
                        <div>Sun</div><div>Mon</div><div>Tue</div><div>Wed</div><div>Thu</div><div>Fri</div><div>Sat</div>
                    </div>
                    <div class="grid grid-cols-7 gap-2">
                        <template v-for="_ in firstDayOfWeek" :key="'empty-'+_">
                            <div></div>
                        </template>
                        <template v-for="day in daysInMonth" :key="day">
                            <div :class="['relative h-16 border rounded-lg flex flex-col items-center justify-start p-1 overflow-hidden', isToday(day) ? 'ring-2 ring-primary border-primary' : '']">
                                <span class="text-xs font-semibold" :class="isToday(day) ? 'text-primary' : ''">{{ day }}</span>
                                <template v-for="event in getEventsForDay(day)" :key="event.title">
                                    <span
                                        class="mt-1 block text-xs text-primary truncate w-full max-w-full cursor-pointer"
                                        style="white-space:nowrap; overflow:hidden; text-overflow:ellipsis;"
                                        :title="event.title"
                                        @click="openEventModal(event)"
                                    >
                                        {{ event.title }}
                                    </span>
                                </template>
                            </div>
                        </template>
                    </div>
                </Card>
                <!-- Events List -->
                <Card class="p-4">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-lg font-semibold">Upcoming Events</h2>
                        <select 
                            v-model="filterRange" 
                            class="px-4 py-2.5 border rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-primary/20 focus:border-primary"
                        >
                            <option :value="7">Next 7 days</option>
                            <option :value="30">Next 30 days</option>
                        </select>
                    </div>
                    <ul class="space-y-4">
                        <li v-for="event in filteredEvents" :key="event.date+event.title+event.time" class="flex flex-col gap-1 border-b pb-2 last:border-b-0 last:pb-0">
                            <div class="flex items-center gap-2 cursor-pointer" @click="openEventModal(event)">
                                <Calendar class="w-4 h-4 text-primary" />
                                <span class="font-medium">{{ event.title }}</span>
                                <span class="ml-auto text-xs text-gray-500">
                                    {{ formatEventDateTime(event) }}
                                </span>
                            </div>
                            <div class="text-xs text-gray-600">{{ event.description }}</div>
                        </li>
                        <li v-if="filteredEvents.length === 0" class="text-center text-gray-400 text-sm">No upcoming events in this range.</li>
                    </ul>
                </Card>
            </div>
            <!-- All Events Section -->
            <Card class="p-4 mt-8">
                <div class="flex flex-col md:flex-row md:items-center md:gap-4 gap-2 mb-4">
                    <h2 class="text-lg font-semibold flex-1">All Events</h2>
                    <div class="flex gap-2 flex-wrap">
                        <div class="relative flex-1 min-w-[200px]">
                            <Search class="absolute left-3 top-1/2 transform -translate-y-1/2 w-4 h-4 text-gray-400" />
                            <input 
                                v-model="allEventsSearch" 
                                type="text" 
                                placeholder="Search events..." 
                                class="w-full pl-10 pr-4 py-2.5 border rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-primary/20 focus:border-primary"
                            />
                        </div>
                        <select 
                            v-model="selectedYear" 
                            class="px-4 py-2.5 border rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-primary/20 focus:border-primary"
                        >
                            <option v-for="year in allYears" :key="year" :value="year">{{ year }}</option>
                        </select>
                        <select 
                            v-model="selectedMonth" 
                            class="px-4 py-2.5 border rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-primary/20 focus:border-primary"
                        >
                            <option :value="null">Whole Year</option>
                            <option v-for="(month, idx) in allMonths" :key="month" :value="idx">{{ month }}</option>
                        </select>
                        <input 
                            v-model="selectedDay" 
                            type="date" 
                            class="px-4 py-2.5 border rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-primary/20 focus:border-primary" 
                            :max="selectedYear + '-' + String((selectedMonth ?? 0) + 1).padStart(2, '0') + '-31'" 
                            placeholder="Day (optional)" 
                        />
                        <button 
                            @click="selectedDay = ''" 
                            v-if="selectedDay" 
                            class="text-xs text-gray-500 underline hover:text-gray-700"
                        >
                            Clear Day
                        </button>
                    </div>
                </div>
                <ul class="space-y-4">
                    <li v-for="(event, index) in paginatedAllEvents" :key="event.date+event.title+event.time" class="flex flex-col gap-1 border-b pb-2 last:border-b-0 last:pb-0">
                        <div class="flex items-center gap-2">
                            <div class="flex items-center gap-2 cursor-pointer flex-1" @click="openEventModal(event)">
                                <Calendar class="w-4 h-4 text-primary" />
                                <span class="font-medium">{{ event.title }}</span>
                                <span class="ml-auto text-xs text-gray-500">
                                    {{ formatEventDateTime(event) }}
                                </span>
                            </div>
                            <div class="relative kebab-menu">
                                <button 
                                    @click.stop="toggleKebabMenu(index)"
                                    class="p-1 hover:bg-gray-100 rounded-full"
                                >
                                    <MoreVertical class="w-4 h-4 text-gray-500" />
                                </button>
                                <div 
                                    v-show="showKebabMenu === index"
                                    class="absolute right-0 mt-1 w-48 bg-white rounded-lg shadow-lg border py-1 z-10"
                                >
                                    <button 
                                        @click="startEditing(event)"
                                        class="w-full px-4 py-2 text-left text-sm hover:bg-gray-100 flex items-center gap-2"
                                    >
                                        <Pencil class="w-4 h-4" /> Edit
                                    </button>
                                    <button 
                                        @click="openDeleteModal(event)"
                                        class="w-full px-4 py-2 text-left text-sm text-red-600 hover:bg-gray-100 flex items-center gap-2"
                                    >
                                        <Trash2 class="w-4 h-4" /> Delete
                                    </button>
                                </div>
                            </div>
                        </div>
                        <div class="text-xs text-gray-600">{{ truncateDescription(event.description) }}</div>
                    </li>
                    <li v-if="paginatedAllEvents.length === 0" class="text-center text-gray-400 text-sm">No events found for this filter.</li>
                </ul>
                <!-- Pagination -->
                <div class="flex justify-between items-center mt-4">
                    <div class="text-sm text-gray-500">
                        Showing {{ (allEventsCurrentPage - 1) * allEventsPerPage + 1 }} to {{ Math.min(allEventsCurrentPage * allEventsPerPage, allEventsFiltered.length) }} of {{ allEventsFiltered.length }} entries
                    </div>
                    <div class="flex gap-2">
                        <Button 
                            size="sm" 
                            variant="outline" 
                            :disabled="allEventsCurrentPage === 1"
                            @click="changeAllEventsPage(allEventsCurrentPage - 1)"
                        >
                            Previous
                        </Button>
                        <Button 
                            size="sm" 
                            variant="outline" 
                            :disabled="allEventsCurrentPage === allEventsTotalPages"
                            @click="changeAllEventsPage(allEventsCurrentPage + 1)"
                        >
                            Next
                        </Button>
                    </div>
                </div>
            </Card>
        </div>
        <!-- Delete Confirmation Modal -->
        <Dialog v-model:open="showDeleteModal">
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>Delete Event</DialogTitle>
                    <DialogDescription>
                        Are you sure you want to delete this event? This action cannot be undone.
                    </DialogDescription>
                </DialogHeader>
                <DialogFooter class="mt-6">
                    <Button variant="outline" type="button" @click="closeDeleteModal">Cancel</Button>
                    <Button variant="destructive" type="button" @click="deleteEvent">Delete</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>

        <!-- Event Details Modal -->
        <Dialog v-model:open="showEventModal">
            <DialogContent v-if="selectedEvent">
                <DialogHeader>
                    <DialogTitle>{{ isEditing ? 'Edit Event' : selectedEvent.title }}</DialogTitle>
                    <DialogDescription v-if="!isEditing">{{ selectedEvent.description }}</DialogDescription>
                </DialogHeader>
                <div v-if="isEditing" class="mt-4 space-y-4">
                    <div>
                        <label class="block text-sm font-medium mb-1">Event Title</label>
                        <input 
                            v-model="selectedEvent.title" 
                            type="text" 
                            class="w-full border rounded px-3 py-2" 
                            placeholder="Enter event title"
                        />
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Description</label>
                        <textarea 
                            v-model="selectedEvent.description" 
                            class="w-full border rounded px-3 py-2"
                            placeholder="Enter event description"
                        ></textarea>
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Date</label>
                        <input 
                            v-model="selectedEvent.date" 
                            type="date" 
                            class="w-full border rounded px-3 py-2"
                            required
                        />
                    </div>
                    <div class="flex gap-2">
                        <div class="flex-1">
                            <label class="block text-sm font-medium mb-1">Time Start</label>
                            <input 
                                v-model="selectedEvent.time" 
                                type="time" 
                                class="w-full border rounded px-3 py-2"
                                required
                            />
                        </div>
                        <div class="flex-1">
                            <label class="block text-sm font-medium mb-1">Time End <span class="text-xs text-gray-400">(optional)</span></label>
                            <input 
                                v-model="selectedEvent.timeEnd" 
                                type="time" 
                                class="w-full border rounded px-3 py-2"
                            />
                        </div>
                    </div>
                </div>
                <div v-else class="mt-4 space-y-2">
                    <div><span class="font-semibold">Date & Time:</span> {{ formatEventDateTime(selectedEvent) }}</div>
                </div>
                <DialogFooter class="mt-6">
                    <Button variant="outline" type="button" @click="closeEventModal">Close</Button>
                    <Button 
                        v-if="isEditing" 
                        type="button" 
                        @click="saveEvent"
                        :disabled="!selectedEvent.title || !selectedEvent.date || !selectedEvent.time"
                    >
                        Save Changes
                    </Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    </AppLayout>
</template>

<style scoped>
.bg-primary {
    background-color: #2563eb;
}
.text-primary {
    color: #2563eb;
}
</style> 