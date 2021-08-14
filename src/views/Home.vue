<template>
  <main class="home">
    <aside>
      <div class="alert error" v-if="errors.length > 0">
        <span>Please check the following errors:</span>
        <ul>
          <li v-for="item in errors" :key="item.message">
            {{ item }}
          </li>
        </ul>
      </div>
      <form id="form-mapt">
        <!-- IPv6 -->
        <label for="ipp6pd">IPv6 PD</label>
        <input
          v-model="ipv6pdStr"
          type="text"
          id="ipp6pd"
          name="ipp6pd"
          placeholder="2a0e:0425:xxxx::/48"
          required
        />
        <small>The prefix delegated to your CE.</small>

        <!-- IPv4 -->
        <label for="ipp4p">IPv4 Prefix</label>
        <input
          v-model="ipv4pStr"
          type="text"
          id="ipp4p"
          name="ipp4p"
          placeholder="101.56.208.0/20"
          value="101.56.208.0/20"
          required
        />
        <small>The IPv4 subnet delegated to your CE.</small>

        <details close>
          <summary>Advanced Fields</summary>

          <!-- EA -->
          <label for="ealen">EA Length</label>
          <input
            v-model="eaLen"
            type="number"
            id="ealen"
            name="ealen"
            placeholder="16"
            min="1"
            max="32"
          />
          <small>The Embedded Address length in bit.</small>

          <!-- PSID -->
          <label for="psidlen">PSID Length</label>
          <input
            v-model="psidLen"
            type="number"
            id="psidlen"
            name="psidlen"
            placeholder="4"
            min="0"
            max="10"
          />
          <small>The PSID length in bit.</small>

          <label for="psidoff">PSID Offset</label>
          <input
            v-model="psidOff"
            type="number"
            id="psidoff"
            name="psidoff"
            placeholder="6"
            min="0"
            max="15"
          />
          <small>The PSID offset.</small>
        </details>
      </form>
    </aside>
    <div>
      <MapProp
        :valid="errors.length === 0 && Boolean(ipv6PD)"
        :ipv6-prefix="ipv6PD"
        :ipv4-prefix="ipv4Prefix"
        :ae-len="eaLen"
        :psid-len="psidLen"
      />
            <MapPorts :valid="errors.length === 0 && Boolean(ipv6PD)"
                      :ipv6-prefix="ipv6PD"
                      :ae-len="eaLen"
                      :psid-offset="psidOff"
                      :psid-len="psidLen"
            />
    </div>
  </main>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";
import { Debounce } from "vue-debounce-decorator";
import { IPv4CidrRange, IPv6, IPv6CidrRange, Validator } from "ip-num";
import MapProp from "@/components/MapProp.vue";
import MapPorts from "@/components/MapPorts.vue"; // @ is an alias to /src

@Component({
  components: {
    MapPorts,
    MapProp,
  },
  metaInfo: {
    title: "Sky MAP-T Visualizer",
    // Override template
    titleTemplate: "%s",
  },
})
export default class Home extends Vue {
  //@Prop({ default: "" })
  private ipv6pdStr: string = "";
  // @Prop({ default: "101.56.208.0/20" })
  private ipv4pStr: string = "101.56.208.0/20";

  // @Prop({ default: 16 })
  private eaLen: number = 16;
  // @Prop({ default: 4 })
  private psidLen: number = 4;
  // @Prop({ default: 4 })
  private psidOff: number = 6;

  // @Prop({ default: () => [] })
  private errors: string[] = [];

  private ipv6PD: IPv6CidrRange | null;
  private ipv4Prefix: IPv4CidrRange | null;

  constructor() {
    super();
    this.ipv6PD = null;
    this.ipv4Prefix = null;
  }

  @Watch("ipv6pdStr")
  @Watch("ipv4pStr")
  @Watch("eaLen")
  @Watch("psidLen")
  @Watch("psidOff")
  @Debounce(800)
  dataChanged() {
    // Cleanup errors
    this.errors.splice(0, this.errors.length);

    const [resultV4, errorsV4] = Validator.isValidIPv4CidrNotation(
      this.ipv4pStr
    );
    this.errors.push(...errorsV4);

    const [resultV6, errorsV6] = Validator.isValidIPv6CidrRange(
      this.ipv6pdStr
    );
    this.errors.push(...errorsV6);

    if (this.errors.length > 0) {
      this.ipv6PD = null;
      this.ipv4Prefix = null;
      return;
    }

    this.ipv6PD = IPv6CidrRange.fromCidr(this.ipv6pdStr);
    this.ipv4Prefix = IPv4CidrRange.fromCidr(this.ipv4pStr);
  }
}
</script>

<style lang="scss">
#form-mapt {
  details {
    border-bottom: 0;
    border-top: var(--border-width) solid var(--accordion-border-color);
    padding-bottom: 0;
    padding-top: calc(var(--spacing) / 2);
  }
}
</style>
