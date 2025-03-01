<script setup lang="ts">
import { Button } from '@/lib/registry/default/ui/button'
import { Command, CommandEmpty, CommandGroup, CommandInput, CommandItem, CommandList, CommandSeparator } from '@/lib/registry/default/ui/command'
import { Dialog, DialogContent } from '@/lib/registry/default/ui/dialog'
import { Toaster as DefaultToaster } from '@/lib/registry/default/ui/toast'
import { Toaster as NewYorkSonner } from '@/lib/registry/new-york/ui/sonner'
import { Toaster as NewYorkToaster } from '@/lib/registry/new-york/ui/toast'
import { TooltipProvider } from '@/lib/registry/new-york/ui/tooltip'

import { useConfigStore } from '@/stores/config'
import { useMagicKeys, useToggle } from '@vueuse/core'
import Circle from '~icons/radix-icons/circle'

import File from '~icons/radix-icons/file'
import RadixIconsGithubLogo from '~icons/radix-icons/github-logo'
import RadixIconsMoon from '~icons/radix-icons/moon'
import RadixIconsSun from '~icons/radix-icons/sun'
import { Content, useData, useRoute, useRouter } from 'vitepress'
import { onMounted, ref, watch } from 'vue'
import CodeConfigCustomizer from '../components/CodeConfigCustomizer.vue'
import Kbd from '../components/Kbd.vue'
import Logo from '../components/Logo.vue'
import MobileNav from '../components/MobileNav.vue'

import ThemePopover from '../components/ThemePopover.vue'
import { docsConfig, type NavItem } from '../config/docs'

const { radius, theme } = useConfigStore()
// Whenever the component is mounted, update the document class list
onMounted(() => {
  document.documentElement.style.setProperty('--radius', `${radius.value}rem`)
  document.documentElement.classList.add(`theme-${theme.value}`)
})

const { frontmatter, isDark } = useData()

const $route = useRoute()
const $router = useRouter()
const toggleDark = useToggle(isDark)

const links = [
  {
    name: 'GitHub',
    href: 'https://github.com/unovue/shadcn-vue',
    icon: RadixIconsGithubLogo,
  },
  // {
  //   name: 'X',
  //   href: 'https://x.com',
  //   icon: TablerBrandX,
  // },
]

const isOpen = ref(false)
const { Meta_K, Ctrl_K } = useMagicKeys({
  passive: false,
  onEventFired(e) {
    if (e.key === 'k' && (e.metaKey || e.ctrlKey))
      e.preventDefault()
  },
})

watch([Meta_K, Ctrl_K], (v) => {
  if (v[0] || v[1])
    isOpen.value = true
})

function handleSelectLink(item: NavItem) {
  if (item.external)
    window.open(item.href, '_blank')
  else
    $router.go(item.href)

  isOpen.value = false
}

watch(() => $route.path, (n) => {
  // @ts-expect-error View Transition API not supported by all the browser yet
  if (document.startViewTransition) {
    // @ts-expect-error View Transition API not supported by all the browser yet
    document.startViewTransition(() => {
      console.log('soft navigating to: ', n)
    })
  }
})
</script>

<template>
  <TooltipProvider>
    <div v-if="$route.data.frontmatter.layout === false">
      <Content :key="$route.path" />
    </div>
    <div v-else vaul-drawer-wrapper class="flex min-h-screen flex-col bg-background">
      <header class="sticky z-40 top-0 bg-background/80 backdrop-blur-lg border-b border-border">
        <div
          class="container flex h-14 max-w-screen-2xl items-center"
        >
          <div class="mr-4 md:mr-1 hidden md:flex">
            <Logo />

            <nav
              class="flex items-center max-lg:space-x-4 space-x-6 text-sm font-medium"
            >
              <a
                v-for="route in docsConfig.mainNav"
                :key="route.title"
                :href="route.href"
                :target="route.external ? '_target' : undefined"
                class="transition-colors hover:text-foreground/80 text-foreground/60"
                :class="{
                  'font-semibold !text-foreground': $route.path === `${route.href}.html`,
                  'hidden lg:block': route?.href?.includes('github'),
                }"
              >
                {{ route.title }}
              </a>
            </nav>
          </div>
          <MobileNav />

          <div class="flex flex-1 items-center justify-between space-x-2 md:justify-end">
            <div class="w-full flex-1 md:w-auto md:flex-none">
              <Button
                variant="outline"
                class="relative h-8 w-full justify-start rounded-[0.5rem] bg-background text-sm font-normal text-muted-foreground shadow-none sm:pr-12 md:w-40 lg:w-64"
                @click="isOpen = true"
              >
                <span class="hidden lg:inline-flex">Search documentation...</span>
                <span class="inline-flex lg:hidden">Search...</span>
                <Kbd class="pointer-events-none absolute right-[0.3rem] top-[0.3rem] hidden h-5 select-none items-center gap-1 rounded border bg-muted px-1.5 font-mono text-[10px] font-medium opacity-100 sm:flex">
                  <span class="text-xs">⌘</span>K
                </Kbd>
              </Button>
            </div>

            <nav class="flex items-center">
              <ThemePopover />

              <CodeConfigCustomizer />

              <Button
                v-for="link in links"
                :key="link.name"
                as="a"
                class="w-9 h-9"
                :href="link.href" target="_blank"
                :variant="'ghost'"
                :size="'icon'"
              >
                <component :is="link.icon" class="w-5 h-5" />
              </Button>

              <ClientOnly>
                <Button
                  class="w-9 h-9"
                  aria-label="Toggle dark mode"
                  :variant="'ghost'"
                  :size="'icon'"
                  @click="toggleDark()"
                >
                  <component
                    :is="isDark ? RadixIconsSun : RadixIconsMoon"
                    class="w-5 h-5 text-foreground"
                  />
                </Button>
              </ClientOnly>
            </nav>
          </div>
        </div>
      </header>

      <div class="flex-1  bg-background">
        <Transition name="fade" mode="out-in">
          <component :is="'docs'" v-if="$route.path.includes('docs')">
            <Transition name="fade" mode="out-in">
              <Content :key="$route.path" />
            </Transition>
          </component>
          <component :is="'examples'" v-else-if="$route.path.includes('examples')">
            <Transition name="fade" mode="out-in">
              <Content :key="$route.path" />
            </Transition>
          </component>
          <component :is="frontmatter.layout" v-else-if="frontmatter.layout">
            <slot />
          </component>
          <main v-else class="container">
            <Transition name="fade" mode="out-in">
              <Content :key="$route.path" />
            </Transition>
          </main>
        </Transition>
      </div>

      <footer class="py-6 md:px-8 md:py-0">
        <div class="container flex flex-col items-center justify-between gap-4 md:h-24 md:flex-row">
          <div class="text-center text-sm leading-loose text-muted-foreground md:text-left">
            <span class="inline-block">
              Built and designed by
              <a
                href="https://twitter.com/shadcn"
                target="_blank"
                class="underline underline-offset-4 font-bold decoration-foreground"
              >
                shadcn
              </a>
            </span>
            <span class="ml-0.5"> . </span>
            <span class="inline-block ml-2">
              Ported to Vue by
              <a
                href="https://github.com/unovue"
                target="_blank"
                class="underline underline-offset-4 font-bold decoration-foreground"
              >
                Radix Vue
              </a>
            </span>
            <span class="ml-0.5"> . </span>
            <span class="inline-block ml-2">
              The code source is available on
              <a
                href="https://github.com/unovue/shadcn-vue"
                target="_blank"
                class="underline underline-offset-4 font-bold decoration-foreground"
              >
                GitHub
              </a>
            </span>
          </div>
        </div>
      </footer>

      <Dialog v-model:open="isOpen">
        <DialogContent class="p-0">
          <Command>
            <CommandInput placeholder="Type a command or search..." />
            <CommandEmpty>
              No results found.
            </CommandEmpty>
            <CommandList
              @escape-key-down=" isOpen = false"
            >
              <CommandGroup heading="Links">
                <CommandItem
                  v-for="item in docsConfig.mainNav"
                  :key="item.title"
                  :heading="item.title"
                  :value="item.title"
                  class="py-3"
                  @select="handleSelectLink(item)"
                >
                  <File class="mr-2 h-5 w-5" />
                  <span>{{ item.title }}</span>
                </CommandItem>
              </CommandGroup>
              <CommandSeparator />
              <CommandGroup v-for="item in docsConfig.sidebarNav" :key="item.title" :heading="item.title">
                <CommandItem
                  v-for="subItem in item.items"
                  :key="subItem.title"
                  :heading="subItem.title"
                  :value="subItem.title"
                  class="py-3"
                  @select="
                    handleSelectLink(subItem)"
                >
                  <Circle class="mr-2 h-4 w-4" />
                  <span>{{ subItem.title }}</span>
                </CommandItem>
              </CommandGroup>
              <CommandSeparator />
              <CommandGroup heading="Theme">
                <CommandItem
                  value="light-theme"
                  class="py-3"
                  @select="
                    () => {
                      isDark = false;
                      isOpen = false;
                    }
                  "
                >
                  <RadixIconsSun class="mr-2 h-5 w-5" />
                  <span>Light Theme</span>
                </CommandItem>
                <CommandItem
                  value="dark-theme"
                  class="py-3"
                  @select="
                    () => {
                      isDark = true;
                      isOpen = false;
                    }
                  "
                >
                  <RadixIconsMoon class="mr-2 h-5 w-5" />
                  <span>Dark Theme</span>
                </CommandItem>
              </CommandGroup>
            </CommandList>
          </Command>
        </DialogContent>
      </Dialog>
      <DefaultToaster />
      <NewYorkSonner class="pointer-events-auto" :theme="'system'" />
      <NewYorkToaster />
    </div>
  </TooltipProvider>
</template>
