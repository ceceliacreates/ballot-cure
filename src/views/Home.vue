<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title
          ><ion-icon :icon="shieldCheckmarkOutline" size="large"></ion-icon
          >BallotCure</ion-title
        >
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">BallotCure</ion-title>
        </ion-toolbar>
      </ion-header>
      <div id="container" class="ion-padding-horizontal">
        <div class="ion-padding">
          <ion-text color="primary">
            <h2>Instructions</h2>
          </ion-text>
          <p>Enter the ballot ID provided in your notice.</p>
        </div>
        <ion-item>
          <ion-label>Enter Ballot ID</ion-label>
          <ion-input
            data-cy="search-input"
            v-model="searchInput"
            type="text"
          ></ion-input>
          <ion-button data-cy="search-button" @click="search"
            >Search</ion-button
          >
        </ion-item>
        <transition name="fade">
          <ion-card v-if="ballot.voterFirstName">
            <ion-card-header>
              <ion-card-title>Ballot Information</ion-card-title>
              <ion-card-subtitle
                >{{ ballot.voterFirstName }}
                {{ ballot.voterLastName }}</ion-card-subtitle
              >
              <ion-card-subtitle>{{ ballot.voterAddress }}</ion-card-subtitle>
            </ion-card-header>
            <ion-card-content>
              <ion-text color="danger"
                ><h2>Issue Requiring Resolution:</h2></ion-text
              >
              <p>{{ ballot.issueDescription }}</p>
            </ion-card-content>
          </ion-card>
        </transition>
        <transition name="fade">
          <div v-if="ballot.voterFirstName">
            <div class="fileUpload ion-padding">
              <span>Click to upload required document</span>
              <ion-input
                data-cy="file-upload"
                type="file"
                name="file"
                v-model="fileName"
                class="upload"
              ></ion-input>
            </div>
            <transition name="fade">
              <div v-if="fileName">
                <p>{{ fileName.slice(12) }}</p>
              </div>
            </transition>
            <ion-button class="ion-margin" @click="submit" size="large"
              >Submit</ion-button
            >
          </div>
        </transition>
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
  IonInput,
  IonLabel,
  IonItem,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonButton,
  IonText,
  IonIcon,
} from "@ionic/vue";
import { shieldCheckmarkOutline } from "ionicons/icons";
import { defineComponent, onMounted, ref, watch, reactive } from "vue";
import "@capacitor-community/http";
import { Plugins } from "@capacitor/core";

export default defineComponent({
  name: "Home",
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonInput,
    IonLabel,
    IonItem,
    IonCard,
    IonCardHeader,
    IonCardTitle,
    IonCardSubtitle,
    IonCardContent,
    IonButton,
    IonText,
    IonIcon,
  },
  setup() {
    return {
      shieldCheckmarkOutline,
    };
  },
  data() {
    return {
      searchInput: "",
      ballotId: "",
      ballot: {},
      fileName: "",
    };
  },
  watch: {
    ballotId(newBallotId, oldBallotId) {
      this.getBallot(newBallotId);
    },
  },
  methods: {
    async fetchBallot(id: string) {
      const { Http } = Plugins;
      const response = await Http.request({
        method: "GET",
        url: `https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots/${id}`,
      });
      const ballot = response.data;
      return ballot;
    },
    async getBallot(id: string) {
      const newBallot = await this.fetchBallot(id);
      this.ballot = newBallot;
    },
    search() {
      if (this.searchInput.length === 5) {
        const newBallotId = this.searchInput;
        this.ballotId = newBallotId;
      }
    },
    async submit() {
      const { Http } = Plugins;
      const ret = await Http.uploadFile({
        url: `https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots/${this.ballotId}`,
        name: this.fileName,
        filePath: this.fileName,
      });
    },
  },
});
</script>

<style scoped>
.fileUpload {
  position: relative;
  overflow: hidden;
  margin: 10px;
  background-color: rgb(174, 174, 175);
  border-radius: 5px;
  color: white;
}
.upload {
  position: absolute;
  top: 0;
  right: 0;
  margin: 0;
  padding: 0;
  font-size: 20px;
  cursor: pointer;
  opacity: 0;
  filter: alpha(opacity=0);
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
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
