<template>
    <div>
        with cloudinary
    </div>

    <button>
        <nuxt-link to="/">
            Go back
        </nuxt-link>
    </button>

    <!-- change formats -->
    <h1>
        Format Changes
    </h1>
    <NuxtImg provider="cloudinary"
             src="v1673946334/cld-sample-5.jpg"
             format="webp"
             sizes="xl:100vw" />
    <h1>
        Size Changes
    </h1>
    <NuxtImg provider="cloudinary"
             src="v1673946334/cld-sample-5.jpg"
             sizes="sm:100vw md:50vw lg:70vw xl:50vw"
             format="webp" />

    <h1>
        Lazy Loading
    </h1>
    <NuxtImg provider="cloudinary"
             src="v1673946334/cld-sample-5.jpg"
             sizes="sm:100vw md:50vw lg:70vw xl:50vw"
             loading="lazy"
             format="webp" />
</template>

<script setup>

</script>

<style lang="scss" scoped></style>