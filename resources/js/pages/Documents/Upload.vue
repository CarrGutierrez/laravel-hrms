<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import { Head } from '@inertiajs/vue3';
import { Upload } from 'lucide-vue-next';

const breadcrumbs = [
    { title: 'Documents', href: '/documents' },
    { title: 'Upload', href: '/documents/upload' },
];
</script>

<template>
    <Head title="Upload Documents" />
    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="flex flex-col gap-6 p-6">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl font-bold flex items-center gap-2">
                    <Upload class="w-6 h-6" /> Upload Documents
                </h1>
            </div>
            <div class="bg-white rounded-lg shadow p-6">
                <p class="text-gray-500">Document upload content will be added here.</p>
            </div>
        </div>
    </AppLayout>
</template> 