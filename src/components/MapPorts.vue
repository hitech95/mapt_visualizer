<template v-if="valid">
  <article class="map-props">
    <header>MAP-T Ports</header>
    <div class="fl">
      <template v-for="(item, index) in portRanges">
        <div class="value" :class="`block-${index}`">{{ item.toString() }}</div>
      </template>
    </div>
    <footer>
      This is a simple calculation based on some reverse-engineering and it can
      be wrong!
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

  constructor(min: number, max: number, reserved: boolean) {
    this.min = min;
    this.max = max;
  }

  inRange(port: number): boolean {
    return this.min <= port && this.max <= port;
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

  private aeBits: BigInteger | null;
  private psidBits: BigInteger | null;

  private portRanges: PortRange[] | null;

  constructor() {
    super();
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
            //const max = (a << this.psidOffset + portsBLen) + this.psidBits?.toJSNumber() + ((1 << portsBLen) - 1)

            return new PortRange(min.toJSNumber(), max.toJSNumber(), a == 0);
          }
        );
      }
    }
  }
}
</script>
