<template>
  <Outer>
    <FetchLoader :state="$fetchState">
      <CrystallizeGrid v-if="grid" :grid="grid" />
      <div v-else class="no-grids">No grids to show on the frontpage</div>
    </FetchLoader>
  </Outer>
</template>

<script>
import { simplyFetchFromGraph } from "../lib/graph";
import fragments from "../lib/graph/fragments";

export default {
  data() {
    return {
      grid: null,
    };
  },
  async fetch() {
    const { locales, locale: code } = this.$i18n;
    const locale = locales.find((l) => l.locale === code) || locales[0];

    const { data } = await simplyFetchFromGraph({
      query: `
        query FRONTPAGE($language: String!, $path: String!) {
          frontpage: catalogue(path: $path, language: $language) {
            ...item
            ...product
          }
        }

        ${fragments}
      `,
      variables: {
        path: "/web-frontpage",
        language: locale.crystallizeCatalogueLanguage,
      },
    });

    // Extract the grid that we want to show
    const grid = data.frontpage?.components?.find(
      (c) => c.type === "gridRelations"
    )?.content?.grids?.[0];

    this.grid = grid;
  },
  head() {
    return {
      title: "Frontpage",
      meta: [
        {
          hid: "description",
          name: "description",
          content: "NuxtJS ecommerce with Crystallize",
        },
      ],
    };
  },
};
</script>

<style scoped>
.no-grids {
  padding: 50px;
  text-align: center;
}
</style>
