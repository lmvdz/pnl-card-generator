

<script setup lang="ts">
import { ref, computed } from 'vue';
import { toast } from 'vue-sonner'

type PositionType = 'LONG' | 'SHORT';

const tradingPair = ref('XBTUSD');
const positionType = ref<PositionType>('SHORT');
const leverage = ref(100);
const currentPrice = ref(89025.2);
const entryPrice = ref(87025.2);
const margin = ref(1000);
const logoUrl = ref('');

const width = ref(350);
const height = ref(200);

const pnlColor = computed(() => {
  const pnl = calculatePnLPercent();
  return pnl >= 0 ? '#2EF2AA' : '#FF4444';
});

const calculatePnLPercent = () => {
  const priceChange = ((currentPrice.value - entryPrice.value) / entryPrice.value) * 100;
  const adjustedPriceChange = positionType.value === 'SHORT' ? -priceChange : priceChange;
  return adjustedPriceChange * leverage.value;
};

const calculatePnLUSD = () => {
  return calculatePnLPercent() / 100  * margin.value;
};

const formatPrice = (price: number) => {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(price);
};

const formatPnL = (pnl: number) => {
  return pnl.toFixed(2);
};

import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import * as z from 'zod'
import { FormField, FormItem, FormLabel, FormControl, FormDescription, FormMessage } from './ui/form';
import { Button } from './ui/button';
import { Input } from './ui/input';

const formSchema = toTypedSchema(z.object({
  tradingPair: z.string().min(2).max(50).default(tradingPair.value),
  positionType: z.enum(['LONG', 'SHORT']).default(positionType.value),
  leverage: z.number().min(1).max(100).default(leverage.value),
  currentPrice: z.number().min(0.000000001).default(currentPrice.value),
  entryPrice: z.number().min(0.000000001).default(entryPrice.value),
  margin: z.number().min(0.0000000001).default(margin.value),
  logoUrl: z.string().url().default(logoUrl.value).optional(),
}))

const form = useForm({
  validationSchema: formSchema,
})

const generatedNewCardSuccess = ref(false);
const generatedNewCardFailure = ref(false);

const onSubmit = form.handleSubmit(async (values) => {
  generatedNewCardSuccess.value = false;
  generatedNewCardFailure.value = false;
  try {
    tradingPair.value = values.tradingPair;
    positionType.value = values.positionType;
    leverage.value = values.leverage;
    currentPrice.value = values.currentPrice;
    entryPrice.value = values.entryPrice;
    margin.value = values.margin;
    generatedNewCardSuccess.value = true;
    setTimeout(() => {
      generatedNewCardSuccess.value = false;
    }, 1000);
    toast.success('Generated new PNL Card');
  } catch(error) {
    generatedNewCardFailure.value = true;
    setTimeout(() => {
      generatedNewCardFailure.value = false;
    }, 1000);
    toast.error('Failed to generate PNL Card');
  }
  
})

const copiedToClipboardSuccess = ref(false);
const copiedToClipboardFailure = ref(false);

const copyToClipboard = () => {
  copiedToClipboardSuccess.value = false;
  copiedToClipboardFailure.value = false;
  const svg = document.querySelector('#cardSvg')!
  const svgData = new XMLSerializer().serializeToString(svg)
  const svgDataUrl = 'data:image/svg+xml;charset=utf-8;base64,' + btoa(unescape(encodeURIComponent(svgData)));

  const image = new Image()

  image.addEventListener('load', () => {
    const canvas = document.createElement('canvas')

    canvas.setAttribute('width', width.value.toString())
    canvas.setAttribute('height', height.value.toString())

    const context = canvas.getContext('2d')!
    context.drawImage(image, 0, 0, width.value, height.value)

    canvas.toBlob((blob) => {
      if (blob) {
        try {
              navigator.clipboard.write([
                  new ClipboardItem({
                      'image/png': blob
                  })
              ]);
              copiedToClipboardSuccess.value = true;
              setTimeout(() => {
                copiedToClipboardSuccess.value = false;
              }, 500);
              toast.success('Copied to clipboard');
          } catch (error) {
              console.error(error);
              copiedToClipboardFailure.value = true;
              setTimeout(() => {
                copiedToClipboardFailure.value = false;
              }, 500);
              toast.error('Failed to copy to clipboard');
          }
      }
    })
  })

  image.src = svgDataUrl

}


const downloadedImageSuccess = ref(false);
const downloadedImageFailure = ref(false);

const downloadImage = () => {
  downloadedImageSuccess.value = false;
  downloadedImageFailure.value = false;
  const svg = document.querySelector('#cardSvg')!
  const svgData = new XMLSerializer().serializeToString(svg)
  const svgDataUrl = 'data:image/svg+xml;charset=utf-8;base64,' + btoa(unescape(encodeURIComponent(svgData)));

  const image = new Image()

  image.addEventListener('load', () => {
    try {
      const canvas = document.createElement('canvas')

      canvas.setAttribute('width', width.value.toString())
      canvas.setAttribute('height', height.value.toString())

      const context = canvas.getContext('2d')!
      context.drawImage(image, 0, 0, width.value, height.value)

      const imgDataUrl = canvas.toDataURL('image/png')

      const a = document.createElement('a');
      a.href = imgDataUrl;
      a.download = `${tradingPair.value}-pnl-card.png`;
      a.click();
      downloadedImageSuccess.value = true;
      setTimeout(() => {
        downloadedImageSuccess.value = false;
      }, 500);
      toast.success('Downloaded Image');
    } catch (error) {
      console.error(error);
      downloadedImageFailure.value = true;
      setTimeout(() => {
        downloadedImageFailure.value = false;
      }, 500);
      toast.error('Failed to download image');
    }
    
  })

  image.src = svgDataUrl
}

</script>


<template>
  <div class="mb-10">
    <!-- {{  form.errors }} -->
    <form @submit="onSubmit" class="w-2/3 space-y-3">
      <FormField v-slot="{ componentField }" name="tradingPair">
        <FormItem>
          <FormLabel>Trading Pair</FormLabel>
          <FormControl>
            <Input type="text" :placeholder="tradingPair" v-bind="componentField" />
          </FormControl>
          <FormDescription>
            This is the trading pair you are trading.
          </FormDescription>
          <FormMessage />
        </FormItem>
      </FormField>
      <FormField v-slot="{ componentField }" name="positionType">
        <FormItem>
          <FormLabel>Position Type</FormLabel>
          <FormControl>
            <Input type="text" :placeholder="positionType" v-bind="componentField" />
          </FormControl>
          <FormDescription>
            Are you long or short?
          </FormDescription>
          <FormMessage />
        </FormItem>
      </FormField>
      <FormField v-slot="{ componentField }" name="leverage">
        <FormItem>
          <FormLabel>Leverage</FormLabel>
          <FormControl>
            <Input type="number" :placeholder="leverage" v-bind="componentField" />
          </FormControl>
          <FormDescription>
            How much leverage are you using?
          </FormDescription>
          <FormMessage /> 
        </FormItem>
      </FormField>
      <FormField v-slot="{ componentField }" name="currentPrice">
        <FormItem>
          <FormLabel>Market Price</FormLabel>
          <FormControl>
            <Input type="number" :placeholder="currentPrice" v-bind="componentField" step="0.01" />
          </FormControl>
          <FormDescription>
            What is the current price of the asset?
          </FormDescription>
          <FormMessage />
        </FormItem>
      </FormField>
      <FormField v-slot="{ componentField }" name="entryPrice">
        <FormItem>
          <FormLabel>Entry Price</FormLabel>
          <FormControl>
            <Input type="number" :placeholder="entryPrice" v-bind="componentField" step="0.01" />
          </FormControl>
          <FormDescription>
            What is the price you entered the position at?
          </FormDescription>
          <FormMessage />
        </FormItem>
      </FormField>
      <FormField v-slot="{ componentField }" name="margin">
        <FormItem>
          <FormLabel>Margin</FormLabel>
          <FormControl>
            <Input type="number" :placeholder="margin" v-bind="componentField" step="0.01" />
          </FormControl>
          <FormDescription>
            How much margin did you use to enter the position?
          </FormDescription>
          <FormMessage />
        </FormItem>
      </FormField>
      <!-- button will change color based on success or failure -->
      <Button type="submit" :class="{ 'bg-green-500': generatedNewCardSuccess, 'bg-red-500': generatedNewCardFailure }">Generate</Button>
    </form>
  </div>
  
  <div class="trading-card">
    <svg
      id="cardSvg"
      :width="width"
      :height="height"
      :viewBox="`0 0 ${width} ${height}`"
      xmlns="http://www.w3.org/500/svg"
    >
      <!-- Background with gradient -->
      <defs>
        <linearGradient id="cardGradient" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color: #0a0a0a; stop-opacity: 1" />
          <stop offset="100%" style="stop-color: #1a1a1a; stop-opacity: 1" />
        </linearGradient>
        <!-- Glow filter -->
        <filter id="neonGlow" x="-20%" y="-20%" width="140%" height="140%">
          <feGaussianBlur stdDeviation="2" result="coloredBlur" />
          <feMerge>
            <feMergeNode in="coloredBlur" />
            <feMergeNode in="SourceGraphic" />
          </feMerge>
        </filter>
      </defs>

      <!-- Main card background -->
      <rect
        x="0"
        y="0"
        :width="width"
        :height="height"
        rx="15"
        ry="15"
        fill="url(#cardGradient)"
        stroke="#2EF2AA"
        stroke-width="2"
      />

      <!-- Trading pair -->
      <text
        x="30"
        y="50"
        fill="#2EF2AA"
        font-family="'Arial', sans-serif"
        font-size="24"
        font-weight="bold"
        filter="url(#neonGlow)"
      >
        {{ tradingPair }}
      </text>

      <!-- Position type -->
      <rect
        x="220"
        y="27"
        width="100"
        height="30"
        rx="5"
        ry="5"
        fill="transparent"
        stroke="#2EF2AA"
        stroke-width="1"
      />
      <text
        x="270"
        y="47"
        fill="#2EF2AA"
        font-family="'Arial', sans-serif"
        font-size="14"
        text-anchor="middle"
        filter="url(#neonGlow)"
      >
        {{ positionType }} {{ leverage }}X
      </text>

      <!-- Price value -->
      <text
        x="30"
        y="120"
        :fill="pnlColor"
        font-family="'Arial', sans-serif"
        font-size="48"
        font-weight="bold"
        filter="url(#neonGlow)"
      >
        ${{ formatPrice(calculatePnLUSD()) }}
      </text>

      <!-- Entry and current values -->
      <text
        x="30"
        y="160"
        fill="#888888"
        font-family="'Arial', sans-serif"
        font-size="14"
      >
        ENTRY
      </text>
      <text
        x="30"
        y="180"
        fill="#2EF2AA"
        font-family="'Arial', sans-serif"
        font-size="14"
      >
        {{ formatPrice(entryPrice) }}
      </text>

      <text
        x="150"
        y="160"
        fill="#888888"
        font-family="'Arial', sans-serif"
        font-size="14"
      >
        CURRENT
      </text>
      <text
        x="150"
        y="180"
        fill="#2EF2AA"
        font-family="'Arial', sans-serif"
        font-size="14"
      >
        {{ formatPrice(currentPrice) }}
      </text>

      <!-- PnL Display -->
      <text
        x="270"
        y="160"
        fill="#888888"
        font-family="'Arial', sans-serif"
        font-size="14"
      >
        ROI
      </text>
      <text
        x="270"
        y="180"
        :fill="pnlColor"
        font-family="'Arial', sans-serif"
        font-size="14"
        font-weight="bold"
        filter="url(#neonGlow)"
      >
        {{ formatPnL(calculatePnLPercent()) }}%
      </text>

      <!-- Logo -->
      <image
        v-if="logoUrl"
        x="270"
        y="140"
        width="40"
        height="40"
        :href="logoUrl"
      />
    </svg>
    <br>
    <!-- button will change color based on success or failure -->
     <!-- if success, also change the text to a checkmark -->
      <!-- if failure, also change the text to a cross -->
    <div class="flex gap-2">
      <Button @click="copyToClipboard" :class="{ 'bg-green-500': copiedToClipboardSuccess, 'bg-red-500': copiedToClipboardFailure }">
        <span v-if="copiedToClipboardSuccess">✓</span>
        <span v-else-if="copiedToClipboardFailure">✗</span>
        <span v-else>Copy to Clipboard</span>
      </Button>
      <!-- download image -->
      <Button @click="downloadImage" :class="{ 'bg-green-500': downloadedImageSuccess, 'bg-red-500': downloadedImageFailure }">
        <span v-if="downloadedImageSuccess">✓</span>
        <span v-else-if="downloadedImageFailure">✗</span>
        <span v-else>Download</span>
      </Button>
    </div>
   
  </div>
</template>

<style scoped>
/* .trading-card {
  display: inline-block;
} */
</style>
