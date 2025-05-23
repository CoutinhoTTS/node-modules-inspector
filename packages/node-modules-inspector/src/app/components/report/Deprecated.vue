<script setup lang="ts">
import type { PackageNode } from 'node-modules-tools'
import type { DeprecationType } from '../../state/payload'
import { useRouter } from '#app/composables/router'
import { DisplayDateBadge } from '#components'
import { computed, nextTick } from 'vue'
import { selectedNode } from '../../state/current'
import { filters } from '../../state/filters'
import { getDeprecatedInfo, payloads } from '../../state/payload'

const router = useRouter()

const groups = computed(() => {
  const all: PackageNode[] = []
  const types: Record<DeprecationType, PackageNode[]> = {
    package: [],
    current: [],
    future: [],
  }

  payloads.filtered.packages.forEach((pkg) => {
    const deprecation = getDeprecatedInfo(pkg)
    if (!deprecation)
      return
    all.push(pkg)
    types[deprecation.type].push(pkg)
  })

  return {
    all,
    types,
  }
})

function showGraph(pkg: PackageNode) {
  filters.state.focus = null
  filters.state.why = [pkg.spec]
  selectedNode.value = pkg
  nextTick(() => {
    router.push({ path: '/graph', hash: location.hash })
  })
}

const titleMap = {
  package: 'Deprecated Packages',
  current: 'Current Versions Deprecated',
  future: 'Future Deprecations',
}
</script>

<template>
  <template v-if="groups.all.length">
    <div badge-color-orange flex="~ gap-2 items-center" rounded-lg p2 my2 mt5 px3>
      <div i-ph-warning-duotone flex-none />
      <span>
        Deprecations in packages usually indicate security risks or misbehaviors.
        Please upgrade to the supported versions or migrate to alternatives.
      </span>
    </div>

    <template v-for="[key, pkgs] of Object.entries(groups.types)" :key="key">
      <template v-if="key !== 'any' && pkgs.length">
        <UiSubTitle>
          <span>{{ titleMap[key as keyof typeof titleMap] }}</span>
          <DisplayNumberBadge :number="pkgs.length" rounded-full text-sm color="badge-color-red" />
        </UiSubTitle>
        <div grid="~ cols-minmax-250px gap-4">
          <div
            v-for="pkg of pkgs" :key="pkg.spec"
            border="~ base rounded-lg" bg-glass
            flex="~ col"
            cursor-pointer
            :class="selectedNode === pkg ? 'border-primary ring-4 ring-primary:20' : ''"
            @click="selectedNode = pkg"
          >
            <div flex="~ items-center gap-2" border="b base" px2 py1>
              <h2 font-mono flex-auto pl2>
                <DisplayPackageSpec :pkg="pkg" />
              </h2>
              <button
                p1 rounded-full op-fade hover:bg-active hover:text-primary hover:op100 flex="~ items-center"
                title="Show Graph"
                @click="showGraph(pkg)"
              >
                <div i-ph-graph-duotone text-lg />
              </button>
            </div>
            <DisplayDeprecationMessage
              :pkg="pkg"
              :show-title="false"
              class="bg-transparent!" pointer-events-none
            />
            <div flex="~ justify-between items-end w-full" mt-auto p2>
              <DisplayDateBadge :pkg rounded-full text-xs />
              <DisplayModuleType :pkg text-xs />
            </div>
          </div>
        </div>
      </template>
    </template>
  </template>
  <template v-else>
    <UiEmptyState
      type="checkmark"
      title="No Deprecated Packages"
      message="Great! None of your packages are deprecated"
    />
  </template>
</template>
