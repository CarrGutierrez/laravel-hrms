<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import { Head } from '@inertiajs/vue3';
import { FileText, Upload, Search, Filter, Loader2, File, X, Download, Plus } from 'lucide-vue-next';
import { ref, computed } from 'vue';

const breadcrumbs = [
    { title: 'Documents', href: '/documents' },
];

const files = ref<any[]>([]); // Uploaded files
const uploading = ref(false);
const uploadProgress = ref<{[key: string]: number}>({});
const searchQuery = ref('');
const categoryFilter = ref('');
const dateFilter = ref('');
const promptInput = ref('');
const fileInput = ref<HTMLInputElement | null>(null);
const showUploadModal = ref(false);

const categories = ['Policies', 'Training', 'Memorandum'];
const allDocuments = ref<any[]>([
  // Example data
  { id: 1, name: 'Employee Handbook.pdf', category: 'Policies', date: '2023-01-10', size: '1.2MB', type: 'pdf' },
  { id: 2, name: 'Safety Training 2023.pptx', category: 'Training', date: '2023-03-15', size: '3.4MB', type: 'pptx' },
  { id: 3, name: 'Holiday Memo.docx', category: 'Memorandum', date: '2023-12-01', size: '0.8MB', type: 'docx' },
]);

const filteredDocuments = computed(() => {
  return allDocuments.value.filter(doc => {
    const matchesSearch =
      !searchQuery.value ||
      doc.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      (promptInput.value && doc.name.toLowerCase().includes(promptInput.value.toLowerCase()));
    const matchesCategory = !categoryFilter.value || doc.category === categoryFilter.value;
    const matchesDate = !dateFilter.value || doc.date.startsWith(dateFilter.value);
    return matchesSearch && matchesCategory && matchesDate;
  });
});

const totalDocs = computed(() => allDocuments.value.length);
const docsByCategory = computed(() => {
  const result: Record<string, number> = {};
  categories.forEach(cat => {
    result[cat] = allDocuments.value.filter(doc => doc.category === cat).length;
  });
  return result;
});

function handleFileChange(e: Event) {
  const input = e.target as HTMLInputElement;
  if (!input.files) return;
  for (const file of Array.from(input.files)) {
    if (!validateFile(file)) continue;
    uploadFile(file);
  }
}
function handleDrop(e: DragEvent) {
  e.preventDefault();
  if (!e.dataTransfer) return;
  for (const file of Array.from(e.dataTransfer.files)) {
    if (!validateFile(file)) continue;
    uploadFile(file);
  }
}
function validateFile(file: File) {
  const allowed = ['application/pdf', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', 'application/msword', 'application/vnd.openxmlformats-officedocument.presentationml.presentation'];
  const maxSize = 10 * 1024 * 1024;
  if (!allowed.includes(file.type)) {
    alert('Invalid file type. Only PDF, DOCX, PPTX allowed.');
    return false;
  }
  if (file.size > maxSize) {
    alert('File too large. Max 10MB.');
    return false;
  }
  return true;
}
function uploadFile(file: File) {
  uploading.value = true;
  uploadProgress.value[file.name] = 0;
  // Simulate upload
  const interval = setInterval(() => {
    if (uploadProgress.value[file.name] >= 100) {
      clearInterval(interval);
      uploading.value = false;
      allDocuments.value.unshift({
        id: Date.now(),
        name: file.name,
        category: categoryFilter.value || 'Uncategorized',
        date: new Date().toISOString().slice(0, 10),
        size: (file.size / (1024 * 1024)).toFixed(1) + 'MB',
        type: file.name.split('.').pop(),
      });
      delete uploadProgress.value[file.name];
    } else {
      uploadProgress.value[file.name] += 10 + Math.random() * 10;
      if (uploadProgress.value[file.name] > 100) uploadProgress.value[file.name] = 100;
    }
  }, 200);
}
function removeFile(id: number) {
  allDocuments.value = allDocuments.value.filter(doc => doc.id !== id);
}
function downloadFile(doc: any) {
  alert('Download: ' + doc.name);
}
function openFileDialog() {
  if (fileInput.value && typeof fileInput.value !== 'string') {
    fileInput.value.click();
  }
}
function openUploadModal() {
  showUploadModal.value = true;
}
function closeUploadModal() {
  showUploadModal.value = false;
}
</script>

<template>
  <Head title="Documents" />
  <AppLayout :breadcrumbs="breadcrumbs">
    <div class="p-4 md:p-8 bg-gray-50 min-h-screen">
      <div class="max-w-5xl mx-auto">
        <div class="bg-white/60 backdrop-blur-md border border-white/30 shadow-lg rounded-2xl p-8 flex flex-col gap-6">
          <!-- Header & Quick Stats -->
          <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-2">
            <div class="flex items-center gap-3">
              <div class="p-2 bg-blue-100 rounded-lg">
                <FileText class="w-6 h-6 text-blue-600" />
              </div>
              <div>
                <h2 class="text-2xl font-bold text-gray-900">Documents</h2>
                <p class="text-gray-500 text-sm mt-1">All your company documents in one place. Upload, search, and manage with ease.</p>
              </div>
            </div>
            <div class="flex gap-4 items-center">
              <div class="flex flex-col items-center">
                <span class="text-xs text-gray-500">Total</span>
                <span class="text-lg font-bold text-blue-700">{{ totalDocs }}</span>
              </div>
              <div v-for="cat in categories" :key="cat" class="flex flex-col items-center">
                <span class="text-xs text-gray-500">{{ cat }}</span>
                <span class="text-lg font-bold text-blue-700">{{ docsByCategory[cat] }}</span>
              </div>
              <button @click="openUploadModal" class="ml-2 p-2 rounded-full bg-blue-600 hover:bg-blue-700 focus:ring-2 focus:ring-blue-300 transition flex items-center justify-center group" title="New Document">
                <Plus class="w-5 h-5 text-white group-hover:scale-110 transition-transform" />
                <span class="sr-only">New Document</span>
              </button>
            </div>
          </div>
          <!-- Search & Filters -->
          <div class="flex flex-col md:flex-row md:items-center gap-3 mb-2 bg-blue-50/40 rounded-xl p-4 border border-blue-100">
            <div class="flex-1 flex gap-2">
              <div class="relative w-full">
                <input v-model="searchQuery" type="text" placeholder="Search documents by name or keyword..." class="w-full rounded-lg border border-gray-200 py-2 pl-10 pr-3 text-sm focus:outline-none focus:ring-2 focus:ring-blue-200" />
                <Search class="absolute left-3 top-2.5 w-4 h-4 text-gray-400" />
              </div>
              <select v-model="categoryFilter" class="rounded-lg border border-gray-200 py-2 px-3 text-sm">
                <option value="">All Categories</option>
                <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
              </select>
              <input v-model="dateFilter" type="text" placeholder="Year (e.g. 2023)" class="rounded-lg border border-gray-200 py-2 px-3 text-sm w-28" />
            </div>
            <input v-model="promptInput" type="text" placeholder="Ask something like 'Show me all training documents from 2023'" class="rounded-lg border border-gray-200 py-2 px-3 text-sm flex-1" />
          </div>
          <!-- Upload Modal -->
          <div v-if="showUploadModal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/30">
            <div class="bg-white rounded-2xl shadow-xl p-8 w-full max-w-md relative">
              <button @click="closeUploadModal" class="absolute top-3 right-3 p-2 rounded-full bg-gray-100 hover:bg-gray-200">
                <X class="w-5 h-5 text-gray-500" />
              </button>
              <h3 class="text-lg font-bold mb-2 flex items-center gap-2"><Upload class="w-5 h-5 text-blue-500" /> Upload Document</h3>
              <div
                class="flex flex-col items-center justify-center border-2 border-dashed border-blue-200 rounded-xl bg-blue-50/40 p-8 cursor-pointer hover:bg-blue-100/60 transition"
                @drop.prevent="handleDrop"
                @dragover.prevent
                @mousedown.prevent="openFileDialog"
              >
                <Upload class="w-10 h-10 text-blue-400 mb-2" />
                <span class="text-gray-700 font-medium">Drag & drop files here or <span class="underline text-blue-600">browse</span></span>
                <span class="text-xs text-gray-500 mt-1">PDF, DOCX, PPTX. Max 10MB each.</span>
                <input ref="fileInput" type="file" class="hidden" multiple @change="handleFileChange" />
              </div>
              <div v-if="Object.keys(uploadProgress).length" class="mt-2 space-y-2">
                <div v-for="(progress, name) in uploadProgress" :key="name" class="flex items-center gap-2">
                  <Loader2 class="w-4 h-4 animate-spin text-blue-500" />
                  <span class="text-xs text-gray-700">Uploading {{ name }}</span>
                  <div class="flex-1 bg-blue-100 rounded h-2 mx-2">
                    <div class="bg-blue-500 h-2 rounded transition-all" :style="{ width: progress + '%' }"></div>
                  </div>
                  <span class="text-xs text-gray-500">{{ Math.round(progress) }}%</span>
                </div>
              </div>
            </div>
          </div>
          <!-- Document List -->
          <div class="overflow-x-auto mt-4">
            <table class="min-w-full text-sm rounded-xl overflow-hidden">
              <thead>
                <tr class="bg-blue-50 text-blue-700">
                  <th class="p-2 text-left">Name</th>
                  <th class="p-2 text-left">Category</th>
                  <th class="p-2 text-left">Date</th>
                  <th class="p-2 text-left">Size</th>
                  <th class="p-2 text-left">Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="doc in filteredDocuments" :key="doc.id" class="border-b hover:bg-blue-50/40 transition">
                  <td class="p-2 flex items-center gap-2">
                    <File class="w-5 h-5 text-blue-400" />
                    <span class="font-medium text-gray-900">{{ doc.name }}</span>
                  </td>
                  <td class="p-2">
                    <span class="inline-block px-2 py-0.5 rounded-full text-xs font-semibold" :class="{
                      'bg-blue-100 text-blue-700': doc.category === 'Policies',
                      'bg-green-100 text-green-700': doc.category === 'Training',
                      'bg-yellow-100 text-yellow-700': doc.category === 'Memorandum',
                      'bg-gray-100 text-gray-700': !categories.includes(doc.category)
                    }">{{ doc.category }}</span>
                  </td>
                  <td class="p-2">{{ new Date(doc.date).toLocaleDateString(undefined, { year: 'numeric', month: 'short', day: 'numeric' }) }}</td>
                  <td class="p-2">{{ doc.size }}</td>
                  <td class="p-2 flex gap-2">
                    <button @click="downloadFile(doc)" class="p-1.5 rounded bg-blue-100 hover:bg-blue-200 text-blue-700" title="Download"><Download class="w-4 h-4" /></button>
                    <button @click="removeFile(doc.id)" class="p-1.5 rounded bg-red-100 hover:bg-red-200 text-red-700" title="Delete"><X class="w-4 h-4" /></button>
                  </td>
                </tr>
                <tr v-if="filteredDocuments.length === 0">
                  <td colspan="5" class="text-center text-gray-400 py-8">No documents found.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </AppLayout>
</template> 