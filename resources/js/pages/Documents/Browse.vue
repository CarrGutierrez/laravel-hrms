<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import { Head } from '@inertiajs/vue3';
import { FileText } from 'lucide-vue-next';

const breadcrumbs = [
    { title: 'Documents', href: '/documents' },
    { title: 'Browse', href: '/documents' },
];
</script>

<template>
    <Head title="Browse Documents" />
    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="flex flex-col gap-6 p-6">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl font-bold flex items-center gap-2">
                    <FileText class="w-6 h-6" /> Browse Documents
                </h1>
            </div>
            <div class="bg-white rounded-lg shadow p-6">
                <p class="text-gray-500">Document browsing content will be added here.</p>
            </div>
        </div>
    </AppLayout>
</template> 