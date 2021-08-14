<template>
  <article class="map-props">
    <header>MAP-T Properties</header>
    <div v-if="!valid">
      <h3>Insert the required data on the left</h3>
      <p>You can find the IPv6 PD under your router WAN status Page</p>
    </div>
    <div v-else>
      <div v-if="ipv6Prefix">
        <span class="">IPv6 PD</span>
        <div class="ipv6 grid">
          <template
              v-for="(item, index) in ipv6Prefix.getFirst().getHexadecatet()"
          >
            <div class="separator" v-if="index > 0">:</div>
            <div class="value">{{ item.toString() }}</div>
          </template>

          <div class="separator">/</div>
          <div class="value">{{ ipv6Prefix.getPrefix().toString() }}</div>
        </div>
      </div>

      <div v-if="ipv6Addr">
        <span class="">IPv6 Source Address</span>
        <div class="ipv6 grid">
          <template v-for="item in ipv6Addr.getHexadecatet()">
            <div class="separator" v-if="index > 0">:</div>
            <div class="value">{{ item.toString() }}</div>
          </template>
        </div>
      </div>

      <div v-if="ipv4Addr">
        <span class="">IPv4 Public Address</span>
        <div class="ipv4 grid">
          <template v-for="(item, index) in ipv4Addr.getOctets()">
            <div class="separator" v-if="index > 0">.</div>
            <div class="value">{{ item.toString() }}</div>
          </template>
        </div>
      </div>

      <!--          <span class="">IPV6 Source Addr</span>-->
      <!--          <div class="ipv6 grid">-->
      <!--            <div v-for="item in ipv6Addr.getHexadecatet()" >{{item.toString()}}</div>-->
      <!--          </div>-->

      <div v-if="aeBits || psidBits">
        <div v-if="aeBits">
          <span class="">AE</span>
          <div><span class="">0x{{ aeBits.toString(16) }}</span></div>
        </div>

        <div v-if="psidBits">
          <span class="">PSID</span>
          <div><span class="">0x{{ psidBits.toString(16) }}</span></div>
        </div>
      </div>
    </div>

    <footer>
      This is a simple calculation based on some reverse-engineering and it can
      be wrong!
    </footer>
  </article>
</template>

<script lang="ts">
import {Component, Prop, Vue, Watch} from "vue-property-decorator";
import {IPv4, IPv4CidrRange, IPv6, IPv6CidrRange} from "ip-num";
import {Debounce} from "vue-debounce-decorator";
import bigInt from "big-integer";
import {BigInteger} from "big-integer";

@Component
export default class MapProp extends Vue {
  @Prop() private valid!: boolean;

  @Prop() private ipv6Prefix!: IPv6CidrRange;
  @Prop() private ipv4Prefix!: IPv4CidrRange;

  @Prop() private aeLen!: number;
  @Prop() private psidLen!: number;

  private ipv6Addr: IPv6 | null;
  private ipv4Addr: IPv4 | null;
  private ipv4Suffix: BigInteger | null;
  private aeBits: BigInteger | null;
  private psidBits: BigInteger | null;

  constructor() {
    super();
    this.ipv6Addr = null;
    this.ipv4Addr = null;
    this.ipv4Suffix = null;
    this.aeBits = null;
    this.psidBits = null;
  }

  @Watch("valid")
  @Watch("ipv6Prefix")
  @Watch("ipv4Prefix")
  @Watch("aeLen")
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
      this.ipv4Suffix = this.aeBits.shiftRight(this.psidLen);

      this.ipv4Addr = IPv4.fromBigInteger(
          this.ipv4Prefix
              .getFirst()
              .getValue()
              .and(
                  bigInt(1)
                      .shiftLeft(this.aeLen - this.psidLen)
                      .minus(1)
                      .negate()
              )
              .or(this.ipv4Suffix)
      );

      this.ipv6Addr = IPv6.fromBigInteger(
          this.ipv6Prefix
              .getFirst()
              .getValue()
              .or(
                  // Last 16 bits are always reserved for PSID
                  this.ipv4Addr.getValue().shiftLeft(16)
              )
              .or(this.psidBits)
      );
    }
  }
}
</script>
