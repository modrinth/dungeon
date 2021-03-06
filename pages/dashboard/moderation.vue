<template>
  <DashboardPage>
    <div class="section-header">
      <h3 class="column-grow-1">Mods</h3>
    </div>
    <ModCard
      v-for="(mod, index) in mods"
      :id="mod.id"
      :key="mod.id"
      :author="mod.author"
      :name="mod.title"
      :description="mod.description"
      :latest-version="mod.latest_version"
      :created-at="mod.published"
      :updated-at="mod.updated"
      :downloads="mod.downloads.toString()"
      :icon-url="mod.icon_url"
      :author-url="mod.author_url"
      :page-url="mod.page_url"
      :categories="mod.categories"
      :edit-mode="true"
      :status="mod.status"
      :is-modrinth="true"
    >
      <button
        class="button column approve"
        @click="changeModStatus(mod.id, 'approved', index)"
      >
        Approve
      </button>
      <button
        class="button column reject"
        @click="changeModStatus(mod.id, 'rejected', index)"
      >
        Reject
      </button>
    </ModCard>
    <div class="section-header">
      <h3 class="column-grow-1">Reports</h3>
    </div>
    <div v-for="(report, index) in reports" :key="report.id" class="report">
      <div class="header">
        <h5 class="title">
          Report for {{ report.item_type }}
          <nuxt-link
            :to="report.item_type + '/' + report.item_id.replace(/\W/g, '')"
            >{{ report.item_id }}</nuxt-link
          >
        </h5>
        <p
          v-tooltip="
            $dayjs(report.created).format(
              '[Created at] YYYY-MM-DD [at] HH:mm A'
            )
          "
          class="date"
        >
          Created {{ $dayjs(report.created).fromNow() }}
        </p>
        <button class="delete iconified-button" @click="deleteReport(index)">
          Delete
        </button>
      </div>
      <div
        v-compiled-markdown="report.body"
        v-highlightjs
        class="markdown-body"
      ></div>
    </div>
  </DashboardPage>
</template>

<script>
import axios from 'axios'

import ModCard from '@/components/ProjectCard'
import DashboardPage from '@/components/DashboardPage'

export default {
  components: {
    DashboardPage,
    ModCard,
  },
  async asyncData(data) {
    const config = {
      headers: {
        Authorization: data.$auth.getToken('local')
          ? data.$auth.getToken('local')
          : '',
      },
    }

    const mods = (
      await axios.get(`https://api.modrinth.com/api/v1/moderation/mods`, config)
    ).data

    const reports = (
      await axios.get(`https://api.modrinth.com/api/v1/report`, config)
    ).data

    return {
      mods,
      reports,
    }
  },
  methods: {
    async changeModStatus(id, status, index) {
      const config = {
        headers: {
          Authorization: this.$auth.getToken('local')
            ? this.$auth.getToken('local')
            : '',
        },
      }

      await axios.patch(
        `https://api.modrinth.com/api/v1/mod/${id}`,
        {
          status,
        },
        config
      )

      this.mods.splice(index, 1)
    },
    async deleteReport(index) {
      const config = {
        headers: {
          Authorization: this.$auth.getToken('local')
            ? this.$auth.getToken('local')
            : '',
        },
      }

      await axios.delete(
        `https://api.modrinth.com/api/v1/report/${this.reports[index].id}`,
        config
      )

      this.reports.splice(index, 1)
    },
  },
}
</script>

<style lang="scss" scoped>
.button {
  margin: 0.25rem 0;
}

.report {
  @extend %card-spaced-b;
  padding: var(--spacing-card-sm) var(--spacing-card-lg);

  .header {
    display: flex;
    align-items: center;
    flex-direction: row;

    .title {
      font-size: var(--font-size-lg);
      margin: 0 0.5rem 0 0;
    }

    .iconified-button {
      margin-left: auto;
    }
  }
}
</style>
