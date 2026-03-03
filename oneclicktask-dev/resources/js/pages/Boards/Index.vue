<script setup lang="ts">
import { Head, Link, router } from '@inertiajs/vue3';
import { ref } from 'vue';

import AppLayout from '@/layouts/AppLayout.vue';
import type { BreadcrumbItem } from '@/types';

interface Board {
    id: number;
    name: string;
    description: string | null;
    color: string | null;
    tasks_count: number;
}

const props = defineProps<{
    boards: Board[];
}>();

const name = ref('');
const description = ref('');
const color = ref('#3b82f6');
const submitting = ref(false);
const error = ref<string | null>(null);

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Boards',
        href: '/boards',
    },
];

const createBoard = () => {
    if (!name.value.trim()) {
        error.value = 'Board name is required.';
        return;
    }

    error.value = null;
    submitting.value = true;

    router.post('/boards', {
        name: name.value,
        description: description.value || null,
        color: color.value || null,
    }, {
        onFinish: () => {
            submitting.value = false;
        },
    });
};
</script>

<template>
    <Head title="Boards" />

    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="mx-auto flex w-full max-w-5xl flex-col gap-6 p-6">
            <section class="rounded-lg border bg-card p-4">
                <h1 class="text-xl font-semibold">Create board</h1>
                <p class="mt-1 text-sm text-muted-foreground">
                    Start by creating your first board.
                </p>

                <form class="mt-4 grid gap-3" @submit.prevent="createBoard">
                    <input
                        v-model="name"
                        type="text"
                        placeholder="Board name"
                        class="h-10 rounded-md border px-3"
                    />
                    <textarea
                        v-model="description"
                        rows="3"
                        placeholder="Description (optional)"
                        class="rounded-md border px-3 py-2"
                    />
                    <input
                        v-model="color"
                        type="color"
                        class="h-10 w-20 rounded-md border p-1"
                    />

                    <p v-if="error" class="text-sm text-red-600">{{ error }}</p>

                    <button
                        type="submit"
                        class="inline-flex h-10 w-fit items-center rounded-md bg-primary px-4 text-primary-foreground disabled:opacity-60"
                        :disabled="submitting"
                    >
                        {{ submitting ? 'Creating...' : 'Create board' }}
                    </button>
                </form>
            </section>

            <section class="rounded-lg border bg-card p-4">
                <h2 class="text-lg font-semibold">Your boards</h2>

                <div v-if="!props.boards.length" class="mt-3 text-sm text-muted-foreground">
                    No boards yet.
                </div>

                <ul v-else class="mt-3 grid gap-2">
                    <li
                        v-for="board in props.boards"
                        :key="board.id"
                        class="rounded-md border p-3"
                    >
                        <Link :href="`/boards/${board.id}`" class="font-medium hover:underline">
                            {{ board.name }}
                        </Link>
                        <p v-if="board.description" class="mt-1 text-sm text-muted-foreground">
                            {{ board.description }}
                        </p>
                    </li>
                </ul>
            </section>
        </div>
    </AppLayout>
</template>
