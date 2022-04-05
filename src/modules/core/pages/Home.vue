<template>
  <AppLayout
    :meta-props="metaProps"
    class="Home"
    :class="{ large: $vuetify.breakpoint.smAndUp }"
  >
    <div class="mt-n3 d-flex justify-center">
      <div class="Home__heroImg" />
      <div class="Hero text-center pt-14 pb-2 px-4 mx-4">
        <h1 class="Hero__title">Cestou Tech</h1>
        <v-carousel cycle hide-delimiters height="auto" :show-arrows="false">
          <v-carousel-item v-for="(tagline, i) in taglines" :key="i">
            <h2 class="Hero__subtitle">{{ tagline }}</h2>
          </v-carousel-item>
        </v-carousel>
      </div>
    </div>

    <div
      class="pt-16 text-body-1 secondary--text text--lighten-3"
      style="margin-top: 600px"
    >
      <p class="text-h5 pb-8">
        Cestou Tech je podcast, kde stretneš Slovákov, ktorí budujú úspešné
        alebo nadchádzajúce softvérové (a iné) produkty.
      </p>

      <div class="text-center pb-8">
        <h3 class="Hero__subtitle secondary--text text--lighten-3 mx-auto">
          Čoskoro...
        </h3>
        <div class="text-body-1 secondary--text text--lighten-3">
          <p class="mb-0">Na podcaste sa práve pracuje.</p>
          <p>Prihlás sa na odber a dostaň sa k novým dielom medzi prvými.</p>
        </div>

        <!-- TODO: Migrate to ghost.org (https://ghost.org/vs/substack)  -->
        <div class="text-center">
          <iframe
            src="https://cestoutech.substack.com/embed"
            width="480"
            height="90"
            frameborder="0"
            scrolling="no"
          />
        </div>
      </div>

      <h3>Show</h3>

      <p>
        Kto sú Slováci, ktorí vybudovali udržateľné a profitabilné podniky zo
        softvérových produktov? V každom diely sa zameriame na jeden projekt a
        stretneme ich zakladateľov. Pozrieme sa na ich pribehy, ako a prečo
        začali, a kde sú teraz. Poriadne ich vyžmýkame o ich skúsenosti: Kde
        našli svojich zákazníkov, ako sa im podarilo monetizovať ich projekt,
        alebo kedy sa z bokovky stalo niečo seriózne.
      </p>

      <br />
      <h3>Moderátor</h3>

      <div
        class="d-flex gap-5"
        :class="
          $vuetify.breakpoint.xsOnly
            ? 'flex-column align-center'
            : 'align-start flex-row-reverse'
        "
      >
        <g-image
          src="../assets/imgs/profile-pic-front-white-bg-cc-sq-md.png"
          width="250"
          :class="{ 'mt-n4': $vuetify.breakpoint.smAndUp }"
        />
        <div>
          <p>
            Som Juro Oravec. Biotechnológ, ktorého zlákal softvér. Ako
            programátor som pracoval na produktoch v malých aj veľkých tímoch, v
            startupoch aj v akadémii, v oblastiach výskumu liečiv,
            bioinformatiky, alebo marketingovej automatizácie.
          </p>
          <p>
            Vždy budujem, skúšam, alebo sa učím niečo bokom, a milujem, keď
            vidím podobné nadšenie aj u iných. Keď som bol v Londýne, nikde inde
            som sa necítil tak živo, ako na pive s Indie Hackers komunitou, teda
            ľuďmi, ktorí bokom budujú malé a stredné podniky zo softvérových
            produktov. Mám to šťastie, že aj na Slovensku už zopár zaujímavých
            projektov poznám, a tak som si hovorím, "Prečo nezviditeľniť tieto
            super projekty a indie hackers na Slovensku?"
          </p>
          <p>
            <a href="https://jurora.vc/about"
              >Viac o mne nájdeš na mojom webe</a
            >
          </p>
        </div>
      </div>
    </div>

    <!-- TODO: Add latest episodes  -->
    <!-- <h2 class="pt-16 mt-16 secondary--text text--lighten-3">Latest posts</h2>
    <PostCollection :posts="latestPosts" :small="$vuetify.breakpoint.xsOnly" /> -->
  </AppLayout>
</template>

<page-query>
query getHomeData {
	blogs: allBlogPost(limit: 5) {
    edges {
      node {
        postId
        path
        title
        description
        datePublished
        images {
          path
          alt
        }
      }
    }
  }
	projects: allProjectPost(limit: 5) {
    edges {
      node {
        postId
        path
        title
        description
        datePublished
        images {
          path
          alt
        }
      }
    }
  }
  metadata {
    siteDescription
  }
}
</page-query>

<script lang="ts">
import { computed, defineComponent } from '@vue/composition-api';
import sortBy from 'lodash/sortBy';
import shuffle from 'lodash/shuffle';
import { VCarousel, VCarouselItem } from 'vuetify/lib/components';

import AppLayout from '@/modules/core/components/AppLayout.vue';
import type { GqlgetHomeDataQuery } from '@/__generated__/graphql';
import type { MetaProps } from '@/modules/core/components/Meta.vue';
import { usePageQuery } from '@/modules/core/utils/useGridsomeQuery';
import PostCollection from '@/modules/post/components/PostCollection.vue';
import { refToRefs } from '../utils/vueReactivity';
import { isNotNil } from '../utils/isNotNull';

const taglines = shuffle([
  'Podcast o slovenských indie hackers',
  'Podcast o slovenských techpreneurs',
  'Podcast o digitálnych produktoch a ich tvorcoch',
]);

const Home = defineComponent({
  name: 'Home',
  components: {
    AppLayout,
    PostCollection,
    VCarousel,
    VCarouselItem,
  },
  setup() {
    const data = usePageQuery<GqlgetHomeDataQuery, GqlgetHomeDataQuery>(
      (data) => data ?? {},
    );
    const { metadata, projects, blogs } = refToRefs(data);

    const latestPosts = computed(() => {
      const posts = [
        ...(projects?.value?.edges ?? []),
        ...(blogs?.value?.edges ?? []),
      ]
        .map((post) => post?.node ?? null)
        .filter(isNotNil)
        .slice(0, 3);

      return sortBy(posts, (post) => post.datePublished);
    });

    const siteDescription = computed(
      (): string => metadata?.value?.siteDescription ?? '',
    );

    const metaProps = computed(
      (): MetaProps => ({
        pageTitle: '',
        pageDescription: siteDescription.value,
      }),
    );

    return {
      latestPosts,
      metaProps,
      taglines,
    };
  },
});
export default Home;
</script>

<style lang="scss">
/* See https://blog.logrocket.com/creating-infinite-css-background-image-loop/ */
@keyframes slide {
  0% {
    transform: translate(0);
  }
  100% {
    transform: translate(-200px); /* The image width */
  }
}

.Home {
  &__heroImg {
    position: absolute;
    max-width: unset;
    min-height: 600px;
    width: 400%;
    background-image: url('~@/modules/core/assets/imgs/cestou-tech-logo-pattern.png');
    background-size: 200px;
    background-repeat: repeat;
    animation: slide 12s linear infinite;
  }

  .Hero {
    position: absolute;
    margin-top: 190px;
    background: white;
    border: 5px solid #363636;

    &__title {
      letter-spacing: 5px !important;
      font-family: 'Avenir', 'Helvetica', 'Helvetica Neue', 'Arial', sans-serif !important;
      font-weight: 500 !important;
      font-size: 4rem !important;
      color: #363636;
    }

    &__subtitle {
      font-family: 'Avenir', 'Helvetica', 'Helvetica Neue', 'Arial', sans-serif !important;
      font-weight: 500 !important;
      color: #363636;
    }
  }

  @media (max-width: 360px) {
    .Hero {
      padding-top: 165px;
    }
  }

  &.large {
    .Hero {
      &__subtitle {
        width: 550px;
      }
    }
  }
}
</style>
