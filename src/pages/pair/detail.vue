<template>
  <div v-if="meta.pair" class="page">
    <template v-if="meta.isAdded">
      <v-tabs-items v-model="tabIndex" touchless>
        <v-tab-item>
          <account-panel :pair="meta.pair" />
        </v-tab-item>
        <v-tab-item>
          <market-panel :pair="meta.pair" />
        </v-tab-item>
      </v-tabs-items>
    </template>
    <market-panel v-else :pair="meta.pair" />

    <page-bottom-action
      :added="meta.isAdded"
      :pair="meta.pair"
      @remove="handleRemove"
      @add="handleAdd"
    />
  </div>
  <div v-else>
    <empty-place-holder />
  </div>
</template>

<script lang="ts">
import {
  Component,
  Mixins,
  ProvideReactive,
  Provide,
  Watch,
} from "vue-property-decorator";
import mixins from "@/mixins";
import { GlobalGetters } from "@/store/types";
import MarketPanel from "@/components/pair/MarketPanel.vue";
import AccountPanel from "@/components/pair/AccountPanel.vue";
import PageBottomAction from "@/components/pair/page-bottom-action/Index.vue";
import PageTabs from "@/components/pair/PageTabs.vue";
import EmptyPlaceHolder from "@/components/particles/EmptyPlaceHolder.vue";
import { Sync } from "vuex-pathify";

@Component({
  components: {
    MarketPanel,
    AccountPanel,
    PageBottomAction,
    EmptyPlaceHolder,
  },
})
class PairDetailPage extends Mixins(mixins.page) {
  @Sync("page/pairDetail@tabIndex") tabIndex!: number;

  @ProvideReactive()
  expand1 = false;

  @ProvideReactive()
  expand2 = false;

  @Provide()
  togggleExpand1() {
    this.expand1 = !this.expand1;
  }

  @Provide()
  togggleExpand2() {
    this.expand2 = !this.expand2;
  }

  get title() {
    return this.meta.symbol;
  }

  get appbar() {
    if (this.meta.isAdded) {
      return {
        align: "center",
        extensionHight: 56,
        extension: this.$createElement(PageTabs),
      };
    }

    return { align: "center" };
  }

  get meta() {
    const getters = this.$store.getters;
    const getPair = getters[GlobalGetters.GET_PAIR_BY_IDS];
    const getIsLiquidityAdded = getters[GlobalGetters.GET_IS_LIQUIDITY_ADDED];
    const getPairMeta = this.$utils.pair.getPairMeta;

    const id1 = this.$route.query.base;
    const id2 = this.$route.query.quote;
    const pair = getPair(id1, id2);
    const pairMeta = getPairMeta(this, pair);
    const isAdded = getIsLiquidityAdded(this, pair);

    return { pair, symbol: pairMeta?.symbol ?? "", isAdded };
  }

  mounted() {
    const source = this.$route.query.source;
    if (source === "market") {
      this.tabIndex = 1;
    } else if (source === "profile") {
      this.tabIndex = 0;
    }
  }

  @Watch("meta.pair", { immediate: true })
  handlePairChange() {
    this.setPageConfig();
  }

  @Watch("tabIndex")
  handleTabIndexChange() {
    if (this.tabIndex === 0) {
      this.handleUpdateUrlSource("profile");
    } else {
      this.handleUpdateUrlSource("market");
    }
  }

  handleUpdateUrlSource(source) {
    this.$router
      .replace({ query: { ...this.$route.query, source } })
      .catch(() => {
        // ignore
      });
  }

  handleRemove() {
    this.$router.push({
      name: "liquidity-remove",
      query: {
        liquidity_asset: this.meta.pair?.liquidity_asset_id,
      },
    });
  }

  handleAdd() {
    this.$router.push({
      name: "liquidity-add",
      query: {
        base: this.meta.pair?.base_asset_id,
        quote: this.meta.pair?.quote_asset_id,
      },
    });
  }
}
export default PairDetailPage;
</script>

<style lang="scss" scoped>
.page {
  padding-bottom: 160px;
}

::v-deep {
  .v-window {
    overflow: visible;
  }
}
</style>
