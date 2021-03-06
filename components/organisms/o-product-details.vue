<template>
  <div class="o-product-details" itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    <meta itemprop="priceCurrency" :content="$store.state.storeView.i18n.currencyCode">
    <meta itemprop="price" :content="parseFloat(product.price_incl_tax).toFixed(2)">
    <meta itemprop="availability" :content="availability">
    <meta itemprop="url" :content="product.url_path">
    <MProductGallery
      :offline-image="offlineImage"
      :gallery="gallery"
      :configuration="productConfiguration"
    />
    <div class="description">
      <SfSticky>
        <MProductShortInfo
          :product="product"
          :custom-options="productCustomOptions"
          :reviews="reviews"
        />
        <ATextAction
          v-if="sizeOption"
          class="text-action"
          :text="$t('Size guide')"
          @click="openSizeGuide"
        />
        <MProductOptionsConfigurable
          v-if="product.type_id =='configurable'"
          :product="product"
          :configuration="productConfiguration"
        />
        <MProductOptionsGroup
          v-if="product.type_id =='grouped'"
          :product-options="product.product_links"
        />
        <MProductOptionsBundle
          v-if="product.bundle_options && product.bundle_options.length > 0"
          :product="product"
        />
        <MProductOptionsCustom
          v-else-if="product.custom_options && product.custom_options.length > 0"
          :product="product"
        />
        <MProductCallToAction
          class="section"
          :product="product"
          :stock="productStock"
        />
        <MProductAdditionalInfo
          :product="product"
          :reviews="reviews"
          :attributes="productAttributes"
        />
      </SfSticky>
    </div>
  </div>
</template>
<script>
import get from 'lodash-es/get'
import config from 'config';
import { mapGetters } from 'vuex';
import { SfSticky } from '@storefront-ui/vue';
import ATextAction from 'theme/components/atoms/a-text-action';
import MProductGallery from 'theme/components/molecules/m-product-gallery';
import MProductShortInfo from 'theme/components/molecules/m-product-short-info';
import MProductCallToAction from 'theme/components/molecules/m-product-call-to-action';
import MProductAdditionalInfo from 'theme/components/molecules/m-product-additional-info';
import MProductOptionsConfigurable from 'theme/components/molecules/m-product-options-configurable';
import MProductOptionsBundle from 'theme/components/molecules/m-product-options-bundle';
import MProductOptionsCustom from 'theme/components/molecules/m-product-options-custom';
import MProductOptionsGroup from 'theme/components/molecules/m-product-options-group';

export default {
  components: {
    SfSticky,
    ATextAction,
    MProductGallery,
    MProductShortInfo,
    MProductCallToAction,
    MProductAdditionalInfo,
    MProductOptionsConfigurable,
    MProductOptionsBundle,
    MProductOptionsCustom,
    MProductOptionsGroup
  },
  props: {
    product: {
      type: Object,
      default: () => ({})
    },
    productGallery: {
      type: Array,
      default: () => []
    },
    productConfiguration: {
      type: Object,
      default: () => ({})
    },
    productCustomOptions: {
      type: Object,
      default: () => ({})
    },
    productAttributes: {
      type: Array,
      default: () => []
    },
    productStock: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    offlineImage () {
      const width = config.products.thumbnails.width;
      const height = config.products.thumbnails.height;
      return {
        mobile: {
          url: this.getThumbnail(this.product.image, width, height),
          alt: this.product.name
        },
        desktop: {
          url: this.getThumbnail(this.product.image, width, height),
          alt: this.product.name
        }
      };
    },
    gallery () {
      const tempImages = [
        'https://shopware-2.vuestorefront.io/media/1c/3e/cb/1575554539/fashion_ZA029207_006_p2_5.jpg.jpg',
        'https://shopware-2.vuestorefront.io/media/87/4c/37/1575554541/fashion_ZA029207_006_p3_5.jpg.jpg',
        'https://shopware-2.vuestorefront.io/media/26/78/a3/1575554536/fashion_ZA029207_006_p1_5.jpg.jpg',
        'https://shopware-2.vuestorefront.io/media/80/01/6a/1575554531/fashion_za029207_008_p.jpg.jpg',
        'https://shopware-2.vuestorefront.io/media/5a/8e/6e/1575554534/fashion_ZA029207_006_p_5.jpg.jpg'
      ].map((src, index) => ({
        id: {color: index, size: index},
        src,
        loading: src
      }))
      const productGallery = this.product.name === 'Inez Full Zip Jacket'
        ? tempImages
        : this.productGallery
      return productGallery.map(imageObject => ({
        id: imageObject.id,
        mobile: {
          url: imageObject.src,
          alt: this.product.name
        },
        desktop: {
          url: imageObject.src,
          alt: this.product.name
        }
      }));
    },
    reviews () {
      const baseReviews = get(this.$store.state.review, 'items.items', [])
      return baseReviews.map((review) => ({
        author: review.nickname,
        date: review.created_at,
        message: `${review.title}: ${review.detail}`,
        rating: 1 // TODO: remove hardcode
      }))
    },
    availability () {
      return this.product.stock && this.product.stock.is_in_stock ? 'InStock' : 'OutOfStock'
    },
    sizeOption () {
      return get(this.productConfiguration, 'size', false)
    }
  },
  methods: {
    openSizeGuide () {
      this.$bus.$emit('modal-show', 'modal-sizeguide');
    }
  }
};
</script>
<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

.o-product-details {
  @include for-desktop {
    display: flex;
  }
}
.description {
  flex: 1;
  padding: 0 var(--spacer-big);
  @include for-desktop {
    margin-left: calc(var(--spacer-big) * 5);
  }
}
.text-action {
  @include for-desktop {
    justify-content: flex-end;
  }
}
.section {
  border-bottom: 1px solid #f1f2f3;
  padding-bottom: 10px;
  @include for-desktop {
    border: 0;
    padding-bottom: 0;
  }
}
</style>
