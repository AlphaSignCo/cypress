<template>
  <div
    data-cy="debug"
    class="flex flex-col gap-16px"
  >
    <div
      data-cy="debug-header"
      class="flex w-full grid py-24px px-24px gap-y-2 items-center overflow-hidden"
    >
      <ul
        data-cy="header-top"
      >
        <li
          v-if="debug?.commitInfo?.summary"
          class="font-medium text-lg text-gray-900 inline"
          :class="{'mr-8px': props.commitsAhead}"
          data-cy="debug-test-summary"
        >
          {{ debug.commitInfo.summary }}
        </li>
        <li
          v-if="props.commitsAhead"
          class="border rounded border-gray-100 h-6 text-sm inline-block"
        >
          <span
            class="font-normal mx-px px-2 text-orange-500 items-center"
            data-cy="debug-commitsAhead"
          >
            {{ t('debugPage.header.commitsAhead', props.commitsAhead) }}
          </span>
        </li>
        <li
          class="text-lg text-gray-400 w-16px inline"
          aria-hidden="true"
        >
          •
        </li>
        <li class="font-normal text-sm text-indigo-500 inline">
          <ExternalLink
            data-cy="debug-header-dashboard-link"
            :href="debug.url || '#'"
          >
            {{ t('debugPage.header.runUrl') }}
          </ExternalLink>
        </li>
      </ul>
      <ul
        data-cy="metadata"
        class="flex flex-wrap font-normal text-sm text-gray-700 gap-x-2 items-center whitespace-nowrap children:flex children:items-center"
      >
        <li
          class="flex flex-row text-sm gap-x-2 items-center justify-center"
        >
          <DebugRunNumber
            v-if="(debug.runNumber && debug.status)"
            :status="debug.status"
            :value="debug.runNumber"
          />
          <DebugResults
            :gql="props.gql"
            data-cy="debug-results"
          />
        </li>
        <li
          v-if="debug?.commitInfo?.branch"
          data-cy="debug-header-branch"
        >
          <i-cy-tech-branch-h_x16 class="mr-1 mr-8px icon-dark-gray-300" />
          <span class="sr-only">Branch Name:</span> {{ debug.commitInfo.branch }}
        </li>
        <li
          v-if="debug.commitInfo?.sha"
          data-cy="debug-header-commitHash"
        >
          <CommitIcon
            class="h-16px fill-white mr-11px w-16px"
          />
          <span class="sr-only">Commit Hash:</span> {{ debug.commitInfo?.sha?.substring(0,7) }}
        </li>
        <li
          v-if="debug?.commitInfo?.authorName"
          data-cy="debug-header-author"
        >
          <UserAvatar
            class="h-16px mr-8px w-16px"
            :email="debug?.commitInfo?.authorEmail"
            data-cy="debug-header-avatar"
          />
          <span class="sr-only">Commit Author:</span> {{ debug.commitInfo.authorName }}
        </li>
        <li
          v-if="debug.createdAt"
          data-cy="debug-header-createdAt"
        >
          <IconTimeStopwatch
            size="16"
            class="mr-9px"
            stroke-color="gray-500"
            fill-color="gray-50"
          />
          <span class="sr-only">Run Total Duration:</span> {{ totalDuration }} ({{ relativeCreatedAt }})
        </li>
      </ul>
    </div>
  </div>
</template>
<script lang="ts" setup>
import { computed } from 'vue'
import DebugResults from './DebugResults.vue'
import ExternalLink from '@cy/gql-components/ExternalLink.vue'
import type { DebugPageHeaderFragment } from '../generated/graphql'
import { IconTimeStopwatch } from '@cypress-design/vue-icon'
import CommitIcon from '~icons/cy/commit_x14'
import { gql } from '@urql/core'
import { dayjs } from '../runs/utils/day.js'
import { useI18n } from 'vue-i18n'
import DebugRunNumber from './DebugRunNumber.vue'
import UserAvatar from '@cy/gql-components/topnav/UserAvatar.vue'

const { t } = useI18n()

gql`
fragment DebugPageHeader on CloudRun {
  id
  runNumber
  createdAt
  status
  totalDuration
  commitInfo {
    sha
  }
  url
  ...RunResults
  commitInfo {
    authorName
    authorEmail
    summary
    branch
  }
}
`

const props = defineProps<{
  gql: DebugPageHeaderFragment
  commitsAhead: number
}>()

const debug = computed(() => props.gql)

const relativeCreatedAt = computed(() => dayjs(new Date(debug.value.createdAt!)).fromNow())

/*
  Format duration to in HH[h] mm[m] ss[s] format. The `totalDuration` field is milliseconds. Remove the leading "00h" if the value is less
  than an hour. Currently, there is no expectation that a run duration will be greater 24 hours or greater, so it is okay that
  this format would "roll-over" in that scenario.
  Ex: 1 second which is 1000ms = 00m 01s
  Ex: 1 hour and 1 second which is 3601000ms = 01h 00m 01s
*/
const totalDuration = computed(() => dayjs.duration(debug.value.totalDuration ?? 0).format('HH[h] mm[m] ss[s]').replace(/^0+h /, ''))

</script>
<style scoped>
[data-cy=metadata] li:not(:first-child)::before {
  content: '•';
  @apply text-lg text-gray-400 pr-8px
}
</style>
