<script>
import { useBranding } from 'shared/composables/useBranding';

const WORKER_BRANDING_URL = 'https://widget-footer-resolver.fteam.workers.dev'

const DEFAULT_BRANDING = {
  brandName: 'Powered By ReplyCleverly',
  logoThumbnail:
    'https://bucket.ping-me.io/widget/1.%20Reply%20Cleverly%20Icon%20(Blue).png',
  widgetBrandURL: 'https://www.replycleverly.com',
};

export default {
  props: {
    disableBranding: {
      type: Boolean,
      default: false,
    },
  },

  setup() {
    const { replaceInstallationName } = useBranding();

    return {
      replaceInstallationName,
    };
  },

  data() {
    return {
      globalConfig: {
        ...DEFAULT_BRANDING,
      },
    };
  },

  computed: {
    brandRedirectURL() {
      try {
        const referrerHost = this.$store.getters['appConfig/getReferrerHost'];
        const url = new URL(this.globalConfig.widgetBrandURL);

        if (referrerHost) {
          url.searchParams.set('utm_source', referrerHost);
          url.searchParams.set('utm_medium', 'widget');
        } else {
          url.searchParams.set('utm_medium', 'survey');
        }

        url.searchParams.set('utm_campaign', 'branding');

        return url.toString();
      } catch (e) {
        return this.globalConfig.widgetBrandURL;
      }
    },
  },

  mounted() {
    this.fetchBranding();
  },

  methods: {
    async fetchBranding() {
      console.log('Calling fetchBranding');
      const websiteToken = window?.chatwootWebChannel?.websiteToken;
      console.log('websiteToken', websiteToken);
      if (!websiteToken) {
        console.log('No website token found');
        return;
      }

      try {
        const response = await fetch(
          `${WORKER_BRANDING_URL}?website_token=${encodeURIComponent(
            websiteToken
          )}`
        );

        if (!response.ok) {
          console.log('Failed to fetch branding', response.statusText);
          return;
        }

        const data = await response.json();
        console.log('data', data);
        this.globalConfig = {
          brandName: data.brandName || DEFAULT_BRANDING.brandName,
          logoThumbnail: data.logoUrl || DEFAULT_BRANDING.logoThumbnail,
          widgetBrandURL: data.brandUrl || DEFAULT_BRANDING.widgetBrandURL,
        };
      } catch (e) {
        this.globalConfig = {
          ...DEFAULT_BRANDING,
        };
      }
    },
  },
};
</script>

<template>
  <div v-if="globalConfig.brandName && !disableBranding" class="px-0 py-3 flex justify-center">
    <a :href="brandRedirectURL" rel="noreferrer noopener nofollow" target="_blank"
      class="branding--link text-n-slate-11 hover:text-n-slate-12 cursor-pointer text-xs inline-flex grayscale-[1] hover:grayscale-0 hover:opacity-100 opacity-90 no-underline justify-center items-center leading-3">
      <img v-if="globalConfig.logoThumbnail" class="ltr:mr-1 rtl:ml-1 max-w-3 max-h-3" :alt="globalConfig.brandName"
        :src="globalConfig.logoThumbnail" />
      <span>
        {{ globalConfig.brandName }}
      </span>
    </a>
  </div>

  <div v-else class="p-3" />
</template>