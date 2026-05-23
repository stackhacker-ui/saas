<script setup lang="ts">
import type { Ref } from "vue";
import { Button } from "~/components/ui/button";
import {
  CommandDialog,
  CommandEmpty,
  CommandGroup,
  CommandInput,
  CommandItem,
  CommandList,
} from "~/components/ui/command";

interface SearchSection {
  id: string;
  title: string;
  titles?: string[];
  content?: string;
}

const props = withDefaults(defineProps<{
  collapsed?: boolean;
}>(), {
  collapsed: false,
});

const searchSections = inject<Ref<SearchSection[] | null>>("searchSections", ref([]));
const open = ref(false);

const items = computed(() => searchSections.value?.map(section => ({
  ...section,
  path: section.id.split("#")[0] || "/docs/getting-started",
  anchor: section.id.includes("#") ? section.id : undefined,
  eyebrow: section.titles?.filter(Boolean).join(" / ") || "Documentation",
})) || []);

function openSearch() {
  open.value = true;
}

async function selectItem(path: string) {
  open.value = false;
  await navigateTo(path);
}

function onKeydown(event: KeyboardEvent) {
  if ((event.metaKey || event.ctrlKey) && event.key.toLowerCase() === "k") {
    event.preventDefault();
    open.value = true;
  }
}

onMounted(() => window.addEventListener("keydown", onKeydown));
onBeforeUnmount(() => window.removeEventListener("keydown", onKeydown));
</script>

<template>
  <Button
    type="button"
    variant="outline"
    class="w-full justify-start text-muted-foreground"
    :class="collapsed ? 'px-2' : ''"
    @click="openSearch"
  >
    <Icon
      name="i-lucide-search"
      class="size-4"
    />
    <span v-if="!props.collapsed">Search documentation</span>
    <kbd
      v-if="!props.collapsed"
      class="ml-auto rounded border px-1.5 py-0.5 text-[10px]"
    >⌘K</kbd>
  </Button>

  <CommandDialog
    v-model:open="open"
    title="Search documentation"
    description="Search documentation pages and sections."
  >
    <CommandInput placeholder="Search documentation..." />
    <CommandList>
      <CommandEmpty>No results found.</CommandEmpty>
      <CommandGroup heading="Documentation">
        <CommandItem
          v-for="item in items"
          :key="item.id"
          :value="`${item.eyebrow} ${item.title} ${item.content || ''}`"
          class="cursor-pointer items-start py-3"
          @select="selectItem(item.anchor || item.path)"
        >
          <Icon
            name="i-lucide-file-text"
            class="mt-0.5 size-4"
          />
          <span class="min-w-0 space-y-1">
            <span class="block truncate font-medium">{{ item.title }}</span>
            <span class="block truncate text-xs text-muted-foreground">{{ item.eyebrow }}</span>
          </span>
        </CommandItem>
      </CommandGroup>
    </CommandList>
  </CommandDialog>
</template>
