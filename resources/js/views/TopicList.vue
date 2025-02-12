<template>
  <section>
    <page-header>
      <template slot="options">
        <div class="dropdown">
          <a
            id="navbarDropdown"
            class="nav-link pr-1"
            href="#"
            role="button"
            data-toggle="dropdown"
            aria-haspopup="true"
            aria-expanded="false"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 24 24"
              width="25"
              class="icon-dots-horizontal"
            >
              <path
                class="fill-light-gray"
                fill-rule="evenodd"
                d="M5 14a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm7 0a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm7 0a2 2 0 1 1 0-4 2 2 0 0 1 0 4z"
              />
            </svg>
          </a>

          <div class="dropdown-menu dropdown-menu-right">
            <router-link :to="{ name: 'create-topic' }" class="dropdown-item">
              {{ trans.new_topic }}
            </router-link>
          </div>
        </div>
      </template>
    </page-header>

    <main class="py-4">
      <div class="col-xl-8 offset-xl-2 col-lg-10 offset-lg-1 col-md-12">
        <div class="my-3">
          <h3 class="mt-3">{{ trans.topics }}</h3>

          <p class="mt-2 text-secondary">
            {{ trans.topics_are_great_for }}
          </p>
        </div>

        <div v-if="isReady" class="mt-5 card shadow-lg">
          <div class="card-body p-0">
            <div :key="`${index}-${topic.id}`" v-for="(topic, index) in topics">
              <router-link
                :to="{
                  name: 'edit-topic',
                  params: { id: topic.id },
                }"
                class="text-decoration-none"
              >
                <div
                  v-hover="{ class: `hover-bg` }"
                  class="p-3"
                  :class="{
                    'border-top': index !== 0,
                    'rounded-top': index === 0,
                    'rounded-bottom': index === topics.length - 1,
                  }"
                >
                  <div class="d-flex align-items-center">
                    <div class="mr-auto pl-2">
                      <p class="mb-0 py-1 lead font-weight-bold">
                        {{ topic.name.it }}
                      </p>
                    </div>
                    <div class="ml-auto d-none d-md-inline-block">
                      <span class="text-secondary mr-3"
                        >{{ suffixedNumber(topic.posts_count) }}
                        {{
                          topic.posts_count == 1 ? trans.post : trans.posts
                        }}</span
                      >
                      <span class="mr-3"
                        >{{ trans.created }}
                        {{
                          moment(topic.created_at).format("MMM D, YYYY")
                        }}</span
                      >
                    </div>

                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      width="25"
                      viewBox="0 0 24 24"
                      class="icon-cheveron-right-circle"
                    >
                      <circle cx="12" cy="12" r="10" style="fill: none" />
                      <path
                        class="fill-light-gray"
                        d="M10.3 8.7a1 1 0 0 1 1.4-1.4l4 4a1 1 0 0 1 0 1.4l-4 4a1 1 0 0 1-1.4-1.4l3.29-3.3-3.3-3.3z"
                      />
                    </svg>
                  </div>
                </div>
              </router-link>
            </div>

            <infinite-loading spinner="spiral" @infinite="fetchTopics">
              <span slot="no-more" />
              <div slot="no-results" class="text-left">
                <div class="my-5">
                  <p class="lead text-center text-muted mt-5">
                    {{ trans.you_have_no_topics }}
                  </p>
                  <p class="lead text-center text-muted mt-1">
                    {{ trans.write_on_the_go }}
                  </p>
                </div>
              </div>
            </infinite-loading>
          </div>
        </div>
      </div>
    </main>
  </section>
</template>

<script>
import { mapGetters } from "vuex";
import Hover from "../directives/Hover";
import InfiniteLoading from "vue-infinite-loading";
import NProgress from "nprogress";
import PageHeader from "../components/PageHeader";
import isEmpty from "lodash/isEmpty";
import strings from "../mixins/strings";

export default {
  name: "topic-list",

  components: {
    InfiniteLoading,
    PageHeader,
  },

  directives: {
    Hover,
  },

  mixins: [strings],

  data() {
    return {
      page: 1,
      topics: [],
      isReady: false,
    };
  },

  computed: {
    ...mapGetters({
      trans: "settings/trans",
    }),
  },

  async created() {
    await Promise.all([this.fetchTopics()]);
    this.isReady = true;
    NProgress.done();
  },

  methods: {
    fetchTopics($state) {
      if ($state) {
        return this.request()
          .get("/api/topics", {
            params: {
              page: this.page,
            },
          })
          .then((response) => {
            if (!isEmpty(response.data) && !isEmpty(response.data.data)) {
              this.page += 1;
              this.topics.push(...response.data.data);

              $state.loaded();
            } else {
              $state.complete();
            }

            if (isEmpty($state)) {
              NProgress.inc();
            }
          })
          .catch(() => {
            NProgress.done();
          });
      }
    },
  },
};
</script>
