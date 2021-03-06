<template>
  <div class="post_id_view">
    <main>
      <article>
        <header>
          <h1>
            {{ post.title }}
          </h1>
          <div class="meta">
            <div class="created_at">
              <PartsSvgIcon :icon="'edit'" :color="'#9e9e9e'" />
              <time :datetime="datetime(post.created_at)">{{ friendlyDatetime(post.created_at) }}</time>
            </div>
            <div class="updated_at" v-if="post.updated_at">
              <PartsSvgIcon :icon="'update'" :color="'#9e9e9e'" />
              <time :datetime="datetime(post.updated_at)">{{ friendlyDatetime(post.updated_at) }}</time>
            </div>
          </div>
          <div class="tags">
            <NuxtLink :to="`/tags/${tag}`" v-for="tag in post.tags" :key="tag">
              {{ urlUnescape(tag) }}
            </NuxtLink>
          </div>
        </header>
        <div id="content" v-html="post.body"></div>
        <footer>
          <!-- (｡･ڡ･｡) -->
        </footer>
      </article>
    </main>
    <ModulesTheSidebar :html="post.body" />
  </div>
</template>

<script setup lang="ts">
import { SITE_FULL_PATH, PostData } from "@/lib/defines"
import { delay, friendlyDatetime, urlUnescape, zeroPadding } from "@/lib/utils"
import secret from "@/secrets"

const router = useRouter()
const slag = ref(router.currentRoute.value.params.slag)

const post = ref(await $fetch<PostData>(`/get-post`, {
  baseURL: secret.API_BASE_URL,
  headers: {
    Authorization: secret.API_KEY,
  },  
  params: {
    slag: slag.value,
  },
}))

onMounted(async () => {
  do {
    await delay(100)
  } while (!document.getElementById("content"))

  useHead({ script: [{ src: "/assets/prism.js", defer: true },] })
})

useHead({ script: [{ src: "/assets/prism.js", defer: true },] })

useSetMeta({
  title: post.value.title,
  description: generateMetaDescription(post.value.body),
  keywords: post.value.tags.join(","),
  url: SITE_FULL_PATH + "/" + slag.value,
  createdAt: post.value.created_at,
  updatedAt: post.value.updated_at,
})

const datetime = (datetime: string) => {
  return `${datetime.slice(0, 4)}-
          ${zeroPadding(Number(datetime.replace(/\d{4}\/(\d\d*)\/.*?$/gmi, "$1")), 2)}-
          ${zeroPadding((Number(datetime.replace(/\d{4}\/\d\d*\/(.*?)$/gmi, "$1"))), 2)}
         `
}

function generateMetaDescription(html: string): string {
  return html  // https://regex101.com/r/ke1Ymn/1
    .replace(/(<.*?>|\n|^\n)/gmi, "")
    .replace(/<h[234].*?>.*?<\/h[234]>/gmi, "")
    .slice(0, 100)
}
</script>

<style lang="scss" scoped>
.post_id_view {
  display: flex;
  main {
    width: calc(100% - var(--width-sidebar));
    padding: 0 47.5px;
    article {
      header {
        h1 {
          @include mobile {
            padding-left: 0em;
          }
        }
        .meta {
          display: flex;
          justify-content: flex-start;
          align-items: center;
          flex-wrap: wrap;
          gap: 1.23em;
          margin-bottom: 0.111em;
          color: #9e9e9e;
          font-size: 0.7em;
          font-weight: bold;
          .created_at, .updated_at {
            position: relative;
            svg {
              top: 1.7px;
              transform: scale(0.95);
            }
            time {
              display: inline-block;
              margin-left: 1.3em;
            }
          }
        }
        .tags {
          a {
            display: inline-block;
            margin-right: 0.7em;
            color: #9e9e9e;
            font-size: 0.777em;
            text-decoration: none;
            word-break: break-all;
            transition: all 0.23s ease-in-out;
            &:hover {
              color: #c27c3e;
              text-decoration: underline;
              &::after {
                color: #9e9e9e;
                text-decoration: none;
              }
            }
            &::after {
              content: ",";
              display: inline-block;
              margin: auto 0em auto 0.1em;
              font-family: "Open Sans";
            }
            &:last-child::after {
              content: none;
            }
          }
        }
      }
    }
    @include tablet {
      padding: 0 13px;
    }
    @include mobile {
      width: 100%;
      padding: 0;
    }
  }
}
.dark {
  .post_id_view {
    main {
      article {
        header {
          .meta,
          .tags {
            color: #a2a2a2;
          }
        }
      }
    }
  }
}
</style>
