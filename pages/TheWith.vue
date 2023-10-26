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
             :placeholder="15"
             format="webp" />

    <h1>
        Defined Height and Width
    </h1>
    <NuxtImg provider="cloudinary"
             src="v1673946334/cld-sample-5.jpg"
             sizes="sm:100vw md:50vw lg:70vw xl:50vw"
             loading="lazy"
             height="600px"
             width="600px"
             format="webp" />

    <h1>
        Quality
    </h1>
    <NuxtImg provider="cloudinary"
             src="v1673946334/cld-sample-5.jpg"
             sizes="sm:100vw md:50vw lg:70vw xl:50vw"
             loading="lazy"
             height="600px"
             width="600px"
             quality="10"
             format="webp" />

    <h1>
        <NuxtImg provider="cloudinary"
                 src="v1673946334/cld-sample-5.jpg"
                 sizes="sm:100vw md:50vw lg:70vw xl:50vw"
                 loading="lazy"
                 height="600px"
                 width="600px"
                 quality="60"
                 format="webp"
                 :modifiers="{ roundCorner: '0:100' }" />

    </h1>
</template>

<script setup>

</script>

<style lang="scss" scoped></style>