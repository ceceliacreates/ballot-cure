<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>BallotCure</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">BallotCure</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container">
        <p>{{ ballot.voterFirstName }} {{ ballot.voterLastName }}</p>
        <p>{{ ballot.issueDescription }}</p>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";
import { defineComponent, onMounted, ref } from "vue";

import "@capacitor-community/http";

import { Plugins } from "@capacitor/core";

async function fetchBallot(ballotId: string) {
  const { Http } = Plugins;

  const response = await Http.request({
    method: "GET",
    url: `https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots/${ballotId}`,
  });

  const ballot = response.data;

  return ballot;
}

export default defineComponent({
  name: "Home",
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
  },
  setup() {
    const ballot = ref({});
    const ballotId = ref("12345");
    const getBallot = async () => {
      ballot.value = await fetchBallot(ballotId.value);
    };

    onMounted(getBallot);

    return {
      ballot,
      getBallot,
    };
  },
});
</script>

<style scoped>
#container {
  text-align: center;

  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
