<template>
  <section>
    <page-header />

    <main class="pt-4">
      <div class="col-xl-8 offset-xl-2 col-lg-10 offset-lg-1 col-md-12 my-3">
        <div class="my-3">
          <h3 class="mt-3">{{ trans.settings }}</h3>
        </div>

        <div v-if="isReady" class="mt-5 card shadow-lg">
          <div class="card-body p-0">
            <div class="d-flex border-top p-3 align-items-center">
              <div class="mr-auto py-1">
                <p class="mb-1 lead font-weight-bold text-truncate">
                  {{ trans.dark_mode }}
                </p>
                <p class="mb-1 d-none d-lg-block text-secondary">
                  {{ trans.toggle_dark_mode }}
                </p>
              </div>
              <div class="ml-auto pl-3">
                <div class="align-middle">
                  <div class="form-group my-auto">
                    <span class="switch switch-sm">
                      <input
                        v-model="darkMode"
                        id="darkMode"
                        type="checkbox"
                        class="switch"
                        :checked="settings.user.dark_mode"
                        @change="toggleDarkMode"
                      />
                      <label for="darkMode" class="mb-0 sr-only">
                        {{ trans.dark_mode }}
                      </label>
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </section>
</template>

<script>
import { mapGetters, mapState } from "vuex";
import Hover from "../directives/Hover";
import NProgress from "nprogress";
import PageHeader from "../components/PageHeader";

export default {
  name: "edit-settings",

  components: {
    PageHeader,
  },

  directives: {
    Hover,
  },

  data() {
    return {
      digest: false,
      locale: null,
      darkMode: false,
      isReady: false,
    };
  },

  computed: {
    ...mapState(["settings"]),
    ...mapGetters({
      trans: "settings/trans",
    }),

    latestRelease() {
      return {
        tag: this.settings.version,
        link: `https://github.com/mtconsultingrouo/canvas-portfolio/releases/tag/${this.settings.version}`,
      };
    },
  },

  created() {
    this.digest = this.settings.user.digest;
    this.locale =
      this.settings.user.locale || this.settings.user.default_locale;
    this.darkMode = this.settings.user.dark_mode;
    this.isReady = true;
    NProgress.done();
  },

  methods: {
    getLocaleDisplayName(locale) {
      let language = require("../data/languages.json")[locale];

      return language.nativeName;
    },

    toggleDigest() {
      this.$store.dispatch("settings/updateDigest", this.digest);
    },

    selectLocale() {
      this.$store.dispatch("settings/updateLocale", this.locale);

      if (this.locale === "ar" || this.locale === "fa") {
        document.body.setAttribute("data-lang", "rtl");
      } else {
        document.body.removeAttribute("data-lang");
      }
    },

    toggleDarkMode() {
      this.$store.dispatch("settings/updateDarkMode", this.darkMode);

      if (this.darkMode === true) {
        document.body.setAttribute("data-theme", "dark");
      } else {
        document.body.removeAttribute("data-theme");
      }
    },
  },
};
</script>
