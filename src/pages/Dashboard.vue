<template>
  <div class="content">
    <div class="md-layout">
      <form>
        <div class="md-layout">
          <div class="md-layout-item md-small-size-100 md-size-50">
            <md-field>
              <label>ISIN</label>
              <md-input type="text" v-model="isin"></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-size-50 text-right">
            <md-button
              class="md-raised md-primary"
              v-on:click="subscribeIsin(isin)"
            >
              Subscribe
            </md-button>
          </div>
        </div>
      </form>
    </div>

    <hr />

    <div class="md-layout">
      <div
        class="md-layout-item md-medium-size-50 md-xsmall-size-100 md-size-25"
        v-for="item in items"
        v-bind:key="item.isin"
        :updateCount="updateCount"
      >
        <md-card class="md-card-stats">
          <md-card-header data-background-color="green">
            {{ item.isin }}
          </md-card-header>

          <md-card-content>
            <p class="category">Price</p>
            <h3 class="title">{{ item.price }}</h3>

            <p class="category">Bid</p>
            <h3 class="title">
              {{ item.bid }}
            </h3>

            <p class="category">Ask</p>
            <h3 class="title">{{ item.ask }}</h3>
          </md-card-content>

          <md-card-actions md-alignment="right">
            <div class="stats">
              <button
                type="button"
                class="md-button md-raised md-warning md-theme-default"
                v-if="item.isActive"
              >
                <div class="md-ripple">
                  <div
                    class="md-button-content"
                    v-on:click="unsubscribeIsin(item.isin)"
                  >
                    Unsubscribe
                  </div>
                </div>
              </button>

              <button
                type="button"
                class="md-button md-raised md-success md-theme-default"
                v-if="!item.isActive"
              >
                <div class="md-ripple">
                  <div
                    class="md-button-content"
                    v-on:click="subscribeIsin(item.isin)"
                  >
                    Subscribe
                  </div>
                </div>
              </button>
            </div>
          </md-card-actions>
        </md-card>
      </div>
    </div>

    <md-snackbar
      :md-position="position"
      :md-duration="isInfinity ? Infinity : duration"
      :md-active.sync="showValidationMsg"
      md-persistent
    >
      <span>Duplicate request</span>
      <md-button class="md-default" @click="showSnackbar = false">
        Okay
      </md-button>
    </md-snackbar>

    <md-snackbar
      :md-position="position"
      :md-duration="isInfinity ? Infinity : duration"
      :md-active.sync="showNetworkErrMsg"
      md-persistent
    >
      <span>
        Unable to get the latest data for the selected stocks due to network
        error. Please reload the UI and retry.
      </span>
      <md-button class="md-default" @click="showSnackbar = false">
        Okay
      </md-button>
    </md-snackbar>
  </div>
</template>

<script>
import { webSocket } from "rxjs/webSocket";

const response = webSocket("ws://159.89.15.214:8080/");
const dataMap = {};

export default {
  mounted() {
    response.subscribe(
      msg => {
        msg.isActive = true;
        dataMap[msg.isin] = msg;
      },
      err => {
        this.showNetworkErrMsg = true;
      },
      () => {}
    );

    this.timer = setInterval(() => {
      this.updateData(dataMap);
    }, 1000);
  },
  data() {
    return {
      items: {},
      isin: "",
      showValidationMsg: false,
      showNetworkErrMsg: false,
      position: "center",
      duration: 3000,
      isInfinity: false,
      timer: "",
      updateCount: 0
    };
  },
  methods: {
    subscribeIsin: function(isin) {
      if (isin) {
        if (this.items[isin] && this.items[isin].isActive !== false) {
          this.showValidationMsg = true;
        } else {
          response.next({
            subscribe: isin
          });
        }
      }
    },
    unsubscribeIsin: function(isin) {
      response.next({
        unsubscribe: isin
      });
      this.items[isin].isActive = false;
      this.setUpdateCount();
    },
    updateData: function(dataMap) {
      if (Object.keys(dataMap).length > 0) {
        this.items = dataMap;
        this.setUpdateCount();
      }
    },
    setUpdateCount: function() {
      this.updateCount += 1;

      if (this.updateData > 50) {
        this.updateCount = 0;
      }
    },
    beforeDestroy: function() {
      clearInterval(this.timer);
    }
  }
};
</script>
