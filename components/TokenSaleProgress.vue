<template>
  <div class="lc-tokensale-progress">
    <div class="lc-tokensale-progress-legend">
      <span class="progress" :style="{ width: `${progress / total * 100}%` }"/>
      <ul>
        <li class="reached"/>
        <li
          v-for="data in points"
          :key="data.value"
          :class="{ reached: progress / total > data.value / points.length }">
          <div>
            <span class="point-legend">
              {{ data.legend }}
            </span>
          </div>
        </li>
      </ul>
    </div>

    <div class="lc-tokensale-progress-bar">
      <span class="progress" :style="{ width: `${progress / total * 100}%` }"/>
      <ul>
        <li class="reached"/>
        <li
          v-for="data in points"
          :key="data.value"
          :class="{ reached: progress / total > data.value / points.length }">
          <div>
            <span class="point-value">
              {{ data.value * 100 }}%
            </span>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>


<script>
import { FINAL_TOKENSALE_ETH_VALUE } from '@/constant';

export default {
  name: 'tokensale-progress',
  props: {
    progress: {
      type: String,
      default: FINAL_TOKENSALE_ETH_VALUE.toFixed(2),
    },
    total: {
      type: String,
      default: '12600',
    },
  },
  data() {
    return {
      points: [
        {
          value: 1,
          legend: '4,200 ETH (Soft Cap)',
        },
        {
          value: 2,
        },
        {
          value: 3,
          legend: '12,600 ETH (Hard Cap)',
        },
      ],
    };
  },
};
</script>


<style lang="scss" scoped>
@import "~assets/variables";

$separator-size: 10px;

.lc-tokensale-progress {
  .lc-tokensale-progress-bar {
    position: relative;
    padding: 0 #{$separator-size / 2 + 4px};
    background-color: $like-gray-3;
    border-radius: 8px;

    .progress {
      border-radius: 8px;
      position: absolute;
      top: 0;
      left: 0;
      bottom: 0;
      background-image: linear-gradient(to left, #62317a, #28646e 10%, #ffdc89);

      width: 0;
      will-change: width;
      transition: width .4s ease-in-out;
    }

    ul > li::after {
      position: absolute;
      top: 50%;
      left: 100%;

      width: $separator-size;
      height: $separator-size;
      margin-top: #{$separator-size / 2 * -1};
      margin-left: #{$separator-size / 2 * -1};

      content: " ";

      border: solid 1px $like-white;
      border-radius: 50%;
      background-color: transparent;
    }
  }

  .lc-tokensale-progress-legend {
    padding: 0 #{$separator-size / 2 + 4px};
    margin-bottom: 4px;

    @media (max-width: 600px) {
      display: none;
    }
  }

  ul {
    display: flex;

    margin: 0;
    padding: 0;

    list-style: none;

    > li {
      position: relative;

      flex: 1;

      &:first-child {
        flex: 0;
      }

      > div {
        > span {
          display: block;

          &.point-value {
            font-weight: 600;
            font-size: 10px;
            margin-right: 8px;
            line-height: 16px;
          }

          &.point-legend {
            font-weight: 600;
            margin-right: 10px;
            margin-left: -10px;
          }

          &.point-value,
          &.point-legend {
            color: #462405;
            text-align: right;
          }
        }
      }

      &.reached {
        &::after {
          background-color: #28646e;
          box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.5);
        }

        .point-value {
          color: white;

          font-weight: bold;
        }

        .point-legend, {
          color: #28646e;
        }
      }
    }
  }
}
</style>
