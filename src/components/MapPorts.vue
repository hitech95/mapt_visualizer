<template>
  <article class="map-props" v-if="valid">
    <header class="flex">
      <div class="title">
        <span>MAP-T Ports</span>
      </div>
      <div class="search">
        <input
          v-model="portFilter"
          type="number"
          min="1"
          max="65535"
          name="portFind"
          placeholder="Validate Port"
          :aria-invalid="
            portFilter ? (!findPort(portFilter) ? 'true' : 'false') : false
          "
        />
      </div>
    </header>
    <section class="ports flex">
      <template v-for="(item, index) in portRanges">
        <div class="element bullet" :class="`block-${index}`" :aria-valid="portFilter && item.inRange(portFilter)">
          {{ item.toString() }}
        </div>
      </template>
    </section>
    <footer>
      <span v-if="!portFilter"
        >Use the search above to find if a port as available</span
      >
      <span class="text-success" v-else-if="portFilter && findPort(portFilter)"
        >Specified port has been found in range
        {{ portRanges.find((el) => el.inRange(portFilter)).toString() }}</span
      >
      <span class="text-error" v-else>The specified port has not been found</span>
    </footer>
  </article>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";
import { IPv6CidrRange } from "ip-num";
import bigInt, { BigInteger } from "big-integer";
import { Debounce } from "vue-debounce-decorator";

interface IPortRange {
  min: number;
  max: number;

  inRange(port: number): boolean;

  toString(): string;
}

export class PortRange implements IPortRange {
  min: number;
  max: number;
  reserved: boolean;

  constructor(min: number, max: number, reserved: boolean) {
    this.min = min;
    this.max = max;
    this.reserved = reserved;
  }

  inRange(port: number): boolean {
    return !this.reserved && this.min <= port && port <= this.max;
  }

  toString(): string {
    return `${this.min}-${this.max}`;
  }
}

@Component
export default class MapPorts extends Vue {
  @Prop() private valid!: boolean;

  @Prop() private ipv6Prefix!: IPv6CidrRange;

  @Prop() private aeLen!: number;
  @Prop() private psidOffset!: number;
  @Prop() private psidLen!: number;

  private portFilter: number | null;

  private aeBits: BigInteger | null;
  private psidBits: BigInteger | null;

  private portRanges: PortRange[] | null;

  constructor() {
    super();
    this.portFilter = null;
    this.aeBits = null;
    this.psidBits = null;
    this.portRanges = [];
  }

  @Watch("valid")
  @Watch("ipv6Prefix")
  @Watch("aeLen")
  @Watch("psidOffset")
  @Watch("psidLen")
  @Debounce(100)
  dataChanged() {
    if (this.valid) {
      this.aeBits = this.ipv6Prefix
        .getFirst()
        .getValue()
        .shiftRight(128 - this.ipv6Prefix.getPrefix().getValue())
        .and(bigInt(1).shiftLeft(this.aeLen).minus(1));

      this.psidBits = this.aeBits.and(
        bigInt(1).shiftLeft(this.psidLen).minus(1)
      );

      if (this.psidBits) {
        const portsBLen = 16 - this.psidOffset - this.psidLen;
        const bits = this.psidBits.shiftLeft(portsBLen);

        this.portRanges = [...Array(1 << this.psidOffset).keys()].map(
          (item, a) => {
            const min = bigInt(a)
              .shiftLeft(this.psidLen + portsBLen)
              .or(bits);
            const max = bigInt(a)
              .shiftLeft(this.psidLen + portsBLen)
              .or(bits)
              .plus(bigInt(1).shiftLeft(portsBLen).minus(1));

            return new PortRange(min.toJSNumber(), max.toJSNumber(), a == 0);
          }
        );
      }
    }
  }

  findPort(port: number): boolean {
    if (!this.portRanges) return false;

    return this.portRanges
      .map((range) => {
        return range.inRange(port);
      })
      .reduce((prev, val) => {
        return prev || val;
      }, false);
  }
}
</script>

<style lang="scss">
.map-props {
  header {
    justify-content: space-between;

    .title {
      display: flex;
      align-items: center;
    }

    .search {
      input {
        margin: 0;
      }
    }
  }

  .ports{
    .element[aria-valid="true"]{
      background: transparent;
      border: 1px solid var(--bullet-active-border-color);
    }
  }
}
</style>
