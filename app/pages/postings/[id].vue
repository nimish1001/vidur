<script setup lang="ts">
const route = useRoute();
const id = route.params.id as string;

const { data: posting } = await usePublicPostingRepository({ id });

if (!posting.value) {
  throw createError({
    statusCode: 404,
    message: 'No such posting found. Maybe the posting has expired.',
  });
}

const { data: careerSiteConfig } = useCareerSiteConfigObjectState();
const companyLogo = useRemoteAsset(careerSiteConfig.value.logo).url;

useHead({
  title: () => `${posting.value?.title + ' | ' || ''}${careerSiteConfig.value.name}`,
});

const tags = computed<string[]>(() => {
  if (posting.value && posting.value.tagsCSV) {
    return posting.value.tagsCSV.split(',').map((t) => t.trim());
  }
  return [];
});

const [firstName] = '';
const [lastName] = '';
const [email] = '';

const isApplying = ref(false);
const appliedSuccessfully = ref(false);
const apply = async () => {
  console.log({ postingId: id, firstName: firstName, lastName: lastName, email: lastName });
  try {
    isApplying.value = true;
    await $fetch('/api/application', {
      method: 'POST',
      body: { postingId: id, firstName: firstName, lastName: lastName, email: lastName },
    });
    console.log({ postingId: id, firstName: firstName, lastName: lastName, email: lastName });
  } catch (e) {
    console.error('Error occured while applying', e);
  } finally {
    isApplying.value = false;
  }
};

const isModalOpen = ref(false);

const open = () => {
  isModalOpen.value = true;
};

const close = () => {
  isModalOpen.value = false;
};

// Google Jobs JSON-LD injection
useJobPostingJsonld(posting);
</script>

<template>
  <main class="grow" v-if="posting">
    <div class="px-4 sm:px-6 lg:px-8 py-8 w-full">
      <!-- Page content -->
      <div class="max-w-5xl mx-auto flex flex-col lg:flex-row lg:space-x-8 xl:space-x-16">
        <!-- Content -->
        <div class="w-full">
          <div class="mb-6">
            <VInputButton as="NuxtLink" variant="outline" to="/">
              <Icon class="fill-current text-zinc-500 mr-2" name="mdi:arrow-left" />
              <span>Back To Jobs</span>
            </VInputButton>
          </div>
          <div class="text-sm text-zinc-500 italic mb-2">Posted {{ timeAgo(new Date(posting.updatedAt)) }}</div>
          <header class="mb-4">
            <!-- Title -->
            <h1 class="text-2xl md:text-3xl text-zinc-800 font-bold">
              {{ posting.title }}
            </h1>
          </header>
          <!-- Company information (mobile) -->
          <div class="bg-white p-5 rounded-2xl border border-zinc-200 mb-6 lg:hidden">
            <div class="text-center mb-6">
              <div class="inline-flex mb-3">
                <img
                  class="w-16 h-16 rounded-full"
                  :src="companyLogo"
                  width="64"
                  height="64"
                  :alt="`${careerSiteConfig.name}'s logo'`"
                />
              </div>
              <div class="text-lg font-bold text-zinc-800 mb-1">
                {{ careerSiteConfig.name }}
              </div>
            </div>
            <div class="space-y-4 sm:flex sm:space-y-0 sm:space-x-2">
              <div class="flex w-full items-center justify-center text-green-500 space-x-2" v-if="appliedSuccessfully">
                <Icon name="teenyicons:tick-circle-solid" class="w-4 h-4" />
                <span>Applied</span>
              </div>
              <VInputButton class="w-full" :disabled="isApplying" v-else>
                Apply Today
                <Icon class="fill-current ml-1" name="mdi:arrow-right" />
              </VInputButton>
            </div>
          </div>

          <!-- Tags -->
          <div class="mb-6" v-if="posting.tagsCSV">
            <div class="flex flex-wrap items-center -m-1">
              <div class="m-1">
                <span
                  class="text-xs inline-flex font-medium bg-zinc-100/30 text-zinc-800 rounded-xl text-center px-2.5 py-1 border border-zinc-500 mr-2"
                  v-for="tag in tags"
                  >{{ tag }}</span
                >
              </div>
            </div>
          </div>
          <hr class="my-6 border-t border-zinc-100" />
          <!-- <Editor :read-only="true" v-model="posting.contents" /> -->
          <div v-html="posting.contents" class="prose" />
        </div>

        <!-- Sidebar -->
        <div class="hidden lg:block space-y-4">
          <!-- Company information (desktop) -->
          <div class="bg-white p-5 rounded-2xl border border-zinc-200 lg:w-72 xl:w-80">
            <div class="text-center mb-6">
              <div class="inline-flex mb-3">
                <img
                  class="w-16 h-16 rounded-full"
                  :src="companyLogo"
                  width="64"
                  height="64"
                  :alt="`${careerSiteConfig.name}'s logo'`"
                />
              </div>
              <div class="text-lg font-bold text-zinc-800 mb-1">
                {{ careerSiteConfig.name }}
              </div>
            </div>
            <div class="space-y-2">
              <div class="flex w-full items-center justify-center text-green-500 space-x-2" v-if="appliedSuccessfully">
                <Icon name="teenyicons:tick-circle-solid" class="w-4 h-4" />
                <span>Applied</span>
              </div>
              <Modal title="Apply">
                <template #input="{ open }">
                  <VInputButton class="w-full" :disabled="isApplying">
                    Apply Today
                    <Icon class="fill-current ml-1" name="mdi:arrow-right" />
                  </VInputButton>
                </template>
                <template #content="{ close }">
                  <div class="max-w-2xl mx-auto bg-white rounded-lg shadow-lg p-6">
                    <form class="space-y-4">
                      <VInputText placeholder="Enter your first name" label="First Name" v-model="firstName" />
                      <VInputText placeholder="Enter your last name" label="Last Name" v-model="lastName" />
                      <VInputText placeholder="Enter your email" label="Email" v-model="email" />

                      <div class="pt-4">
                        <VInputButton class="w-full" @click="apply" :disabled="isApplying">
                          Submit Application
                        </VInputButton>
                      </div>
                    </form>
                  </div>
                </template>
              </Modal>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
