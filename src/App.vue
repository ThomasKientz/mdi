<template>
  <v-app>
    <v-snackbar top color="success" :timeout="1600" v-model="toast">
      <span>Copied to clipboard!</span>
      <template v-slot:action>
        <v-icon style="color: inherit;">mdi-clipboard-check</v-icon>
      </template>
    </v-snackbar>

    <v-app-bar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>MDI</span>
        <span class="font-weight-light">SEARCH</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>

      <v-menu v-model="menu" :close-on-content-click="false" offset-y>
        <template v-slot:activator="{ on }">
          <v-btn icon v-on="on">
            <v-icon>mdi-cog</v-icon>
          </v-btn>
        </template>

        <v-card>
          <v-list>
            <v-list-item>
              <v-list-item-action>
                <v-switch v-model="darkMode" hide-details></v-switch>
              </v-list-item-action>
              <v-list-item-title>Dark mode</v-list-item-title>
            </v-list-item>

            <v-divider></v-divider>

            <v-subheader>Name format</v-subheader>
            <v-list-item>
              <v-radio-group hide-details class="mt-0" v-model="format">
                <v-radio label="account-circle" value="basic"></v-radio>
                <v-radio label="accountCircle" value="camelCase"></v-radio>
                <v-radio
                  label="mdiAccountCircle"
                  value="camelCasePrefixed"
                ></v-radio>
                <v-radio label="mdi-account-circle" value="prefixed"></v-radio>
                <v-radio
                  label="mdi:account-circle"
                  value="colon:prefixed"
                ></v-radio>
              </v-radio-group>
            </v-list-item>
          </v-list>
        </v-card>
      </v-menu>

      <v-btn href="https://github.com/ThomasKientz/mdi" icon>
        <v-icon>mdi-github</v-icon>
      </v-btn>

      <a
        class="ml-3"
        style="color: inherit;"
        href="https://materialdesignicons.com/"
        target="blank"
        >MDI {{ mdiVersion }}</a
      >
    </v-app-bar>

    <v-content>
      <v-container>
        <ais-instant-search
          :index-name="indexName"
          :search-client="searchClient"
        >
          <ais-configure :hits-per-page.camel="100" />
          <ais-search-box>
            <div slot-scope="{ currentRefinement, isSearchStalled, refine }">
              <v-layout align-center>
                <v-text-field
                  prepend-inner-icon="mdi-magnify"
                  label="Search"
                  :value="currentRefinement"
                  @input="refine($event)"
                  :loading="isSearchStalled"
                  autofocus
                  clearable
                />
                <ais-powered-by
                  :theme="$vuetify.theme.dark ? 'dark' : undefined"
                />
              </v-layout>
            </div>
          </ais-search-box>
          <ais-hits>
            <v-row justify="space-between" slot-scope="{ items }" wrap>
              <v-col v-for="item in items" :key="item.objectID">
                <v-tooltip bottom>
                  <template v-slot:activator="{ on }">
                    <v-btn
                      v-on="on"
                      @click="copy(item.name)"
                      class="ma-4"
                      large
                      icon
                      text
                    >
                      <v-icon large>mdi-{{ item.name }}</v-icon>
                    </v-btn>
                  </template>
                  <span>{{ item.name }}</span>
                </v-tooltip>
              </v-col>
              <v-col v-for="n in 10" :key="'placeholder-' + n">
                <v-btn disabled class="ma-4" large icon text>
                  <v-icon class="ma-4" large></v-icon>
                </v-btn>
              </v-col>
            </v-row>
          </ais-hits>
          <ais-pagination>
            <v-pagination
              slot-scope="{
                currentRefinement,
                nbPages,
                pages,
                isFirstPage,
                isLastPage,
                refine
              }"
              :value="currentRefinement + 1"
              @input="refine($event - 1)"
              :length="nbPages"
            />
          </ais-pagination>
          <ais-stats>
            <p
              class="body-2 text-center mt-3"
              slot-scope="{
                hitsPerPage,
                nbPages,
                nbHits,
                page,
                processingTimeMS
              }"
            >
              {{ nbHits }} icons retrieved in {{ processingTimeMS }}ms. Limited
              to 1000 results.
            </p>
          </ais-stats>
        </ais-instant-search>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import {
  AisInstantSearch,
  AisSearchBox,
  AisPagination,
  AisPoweredBy,
  AisStats
} from "vue-instantsearch";
import algoliasearch from "algoliasearch";

export default {
  name: "App",

  components: {
    AisInstantSearch,
    AisSearchBox,
    AisPagination,
    AisPoweredBy,
    AisStats
  },

  data: () => ({
    searchClient: algoliasearch(
      process.env.VUE_APP_ALGOLIA_APP_ID,
      process.env.VUE_APP_ALGOLIA_API_KEY
    ),
    indexName: "MDI",
    toast: false,
    menu: false,
    format: localStorage.getItem("format") || "basic",
    mdiVersion: process.env.VUE_APP_MDI_VERSION
  }),

  computed: {
    darkMode: {
      get() {
        return localStorage.getItem("darkMode") !== "false";
      },
      set(v) {
        localStorage.setItem("darkMode", v);
        this.$vuetify.theme.dark = v;
      }
    }
  },

  watch: {
    format(v) {
      localStorage.setItem("format", v);
    }
  },

  methods: {
    copy(name) {
      const formated =
        this.format == "prefixed"
          ? "mdi-" + name
          : this.format == "camelCase"
          ? toCamelCase(name)
          : this.format == "camelCasePrefixed"
          ? toCamelCase("mdi-" + name)
          : this.format == "colon:prefixed"
          ? "mdi:" + name
          : name;

      this.$copyText(formated).then(() => {
        this.toast = true;
      });
    }
  }
};

const toCamelCase = str => {
  return str.split("-").reduce((res, string, index) => {
    const text =
      index > 0 ? string.charAt(0).toUpperCase() + string.slice(1) : string;
    return res + text;
  });
};
</script>
